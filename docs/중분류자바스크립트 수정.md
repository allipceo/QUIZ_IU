// 서대리 긴급 수정: 기존 JavaScript 부분을 이 코드로 완전 교체

// ACIUApp 네임스페이스 구조 (수정된 버전)
const ACIUApp = {
    // 타이머 관리
    timers: {
        timeUpdate: null,
        quoteUpdate: null,
        ddayUpdate: null
    },

    // 브라우저 호환성 확인
    compatibility: {
        checkBrowserSupport() {
            if (!window.localStorage) {
                ACIUApp.ui.showErrorMessage('localStorage를 지원하지 않는 브라우저입니다.');
                return false;
            }
            return true;
        }
    },

    // 데이터 관리
    data: {
        questions: [],
        userSettings: {
            name: '조대표님',
            examDate: '2025-10-25',
            lastUpdated: new Date().toISOString()
        },
        learningHistory: {
            attempts: {},
            lastUpdated: new Date().toISOString()
        },
        
        // 중분류 학습용 데이터 구조 (노팀장 확정)
        mainCategories: {
            '06재산보험': [
                { name: '화재보험', count: 188 },
                { name: '재물보험재보험', count: 130 },
                { name: '패키지보험', count: 53 }
            ],
            '07특종보험': [
                { name: '미확인', count: 110 },
                { name: '기술보험', count: 93 },
                { name: '범죄보험', count: 51 }
            ],
            '08배상책임보험': [], // 50문제 이상 없음
            '09해상보험': [
                { name: '미확인', count: 165 },
                { name: '해상보험조건과보상범위', count: 57 },
                { name: '해상보험의손해사정', count: 53 }
            ]
        },
        
        otherCategories: {
            '06재산보험': [
                { name: '동산종합보험', count: 15 },
                { name: '기업휴지보험', count: 8 },
                { name: '재물보험', count: 5 }
            ],
            '07특종보험': [
                { name: '기타특종보험', count: 26 },
                { name: '종합보험', count: 8 },
                { name: '개요', count: 4 }
            ],
            '08배상책임보험': [
                { name: '보관자배상책임', count: 49 },
                { name: '전문직업배상', count: 47 },
                { name: '개요', count: 37 },
                { name: '도급업자배상책임', count: 33 },
                { name: '임원배상', count: 32 },
                { name: '시설소유관리자', count: 26 },
                { name: '생산물배상책임', count: 24 },
                { name: '기타주요보험', count: 20 }
            ],
            '09해상보험': [
                { name: '적하보험', count: 42 },
                { name: '해상보험기초', count: 33 },
                { name: '계약의 체결과 보험료', count: 23 },
                { name: '선박보험', count: 23 },
                { name: '운송보험', count: 20 },
                { name: '해상보험조건', count: 4 }
            ]
        },

        // CSV 데이터 로딩
        async loadQuestions() {
            try {
                const response = await fetch('ins_master_db.csv');
                if (!response.ok) {
                    throw new Error(`HTTP error! status: ${response.status}`);
                }
                const csvContent = await response.text();
                
                const parsed = Papa.parse(csvContent, {
                    header: true,
                    dynamicTyping: false,
                    skipEmptyLines: true,
                    delimiter: ',',
                    quoteChar: '"',
                    fastMode: true
                });

                // 필터링: QCODE가 있고 SOURCE가 '인스교재' 또는 '중개사시험'인 경우
                this.questions = parsed.data.filter(row =>
                    row.QCODE && 
                    (row.SOURCE === '인스교재' || row.SOURCE === '중개사시험')
                );

                console.log(`데이터 로딩 완료: ${this.questions.length}개 문제`);
                
                // 데이터 로딩 완료 후 UI 업데이트
                ACIUApp.ui.updateQuestionCounts();
                ACIUApp.ui.updateStatistics();
            } catch (error) {
                console.error('데이터 로딩 실패:', error);
                ACIUApp.ui.showErrorMessage('데이터를 불러오는데 실패했습니다. 파일 경로를 확인해주세요.');
            }
        },

        // 사용자 설정 로딩
        loadUserSettings() {
            try {
                const saved = localStorage.getItem('aciu_user_settings');
                if (saved) {
                    this.userSettings = { ...this.userSettings, ...JSON.parse(saved) };
                }
            } catch (error) {
                console.error('사용자 설정 로딩 실패:', error);
            }
        },

        // 사용자 설정 저장
        saveUserSettings() {
            try {
                localStorage.setItem('aciu_user_settings', JSON.stringify(this.userSettings));
            } catch (error) {
                console.error('사용자 설정 저장 실패:', error);
            }
        },

        // 학습 이력 로딩
        loadLearningHistory() {
            try {
                const saved = localStorage.getItem('aciu_learning_history');
                if (saved) {
                    this.learningHistory = { ...this.learningHistory, ...JSON.parse(saved) };
                }
            } catch (error) {
                console.error('학습 이력 로딩 실패:', error);
            }
        }
    },

    // UI 관리
    ui: {
        // 초기화
        init() {
            this.updateHeader();
            this.updateQuestionCounts();
            this.updateStatistics();
            this.updateQuote();
            this.updateCurrentTime();
            this.updateDdayCount();
        },

        // 헤더 업데이트
        updateHeader() {
            const userInfo = document.getElementById('user-info');
            const examDday = document.getElementById('exam-dday');
            
            if (userInfo) userInfo.textContent = `사용자: ${ACIUApp.data.userSettings.name}`;
            
            const examDate = new Date(ACIUApp.data.userSettings.examDate);
            const today = new Date();
            const diffTime = examDate.getTime() - today.getTime();
            const diffDays = Math.ceil(diffTime / (1000 * 60 * 60 * 24));
            
            if (examDday) examDday.textContent = `D-${Math.max(0, diffDays)}일`;
        },

        // 소스별 문제 수 업데이트
        updateQuestionCounts() {
            const questions = ACIUApp.data.questions;
            
            // 소스별 문제 수 계산
            const insQuestions = questions.filter(q => q.SOURCE === '인스교재').length;
            const examQuestions = questions.filter(q => q.SOURCE === '중개사시험').length;
            const totalQuestions = questions.length;
            
            // UI 업데이트
            const insElement = document.getElementById('ins-questions');
            const examElement = document.getElementById('exam-questions');
            const totalElement = document.getElementById('total-questions');
            
            if (insElement) insElement.textContent = insQuestions.toLocaleString();
            if (examElement) examElement.textContent = examQuestions.toLocaleString();
            if (totalElement) totalElement.textContent = totalQuestions.toLocaleString();
        },

        // 통계 업데이트
        updateStatistics() {
            const totalQuestions = ACIUApp.data.questions.length;
            const solvedQcodes = new Set(Object.keys(ACIUApp.data.learningHistory.attempts));
            const overallRate = totalQuestions > 0 ? Math.round((solvedQcodes.size / totalQuestions) * 100) : 0;
            
            // 일일 진도율
            const today = new Date().toISOString().slice(0, 10);
            const todayData = Object.values(ACIUApp.data.learningHistory.attempts)
                .filter(attempt => attempt.timestamp && attempt.timestamp.slice(0, 10) === today);
            const todaySolved = todayData.length;
            const todayCorrect = todayData.filter(attempt => attempt.correct).length;
            const dailyRate = todaySolved > 0 ? Math.round((todayCorrect / todaySolved) * 100) : 0;
            
            // UI 업데이트
            const completedElement = document.getElementById('completed-questions');
            const overallElement = document.getElementById('overall-progress-rate');
            const dailyElement = document.getElementById('daily-progress-rate');
            
            if (completedElement) completedElement.textContent = solvedQcodes.size.toLocaleString();
            if (overallElement) overallElement.textContent = `${overallRate}%`;
            if (dailyElement) dailyElement.textContent = `${dailyRate}%`;
        },

        // 현재 시간 업데이트
        updateCurrentTime() {
            const now = new Date();
            const timeString = now.toLocaleString('ko-KR', {
                year: 'numeric',
                month: 'long',
                day: 'numeric',
                weekday: 'long',
                hour: '2-digit',
                minute: '2-digit',
                second: '2-digit',
                hour12: true
            });
            const timeElement = document.getElementById('current-time');
            if (timeElement) timeElement.textContent = timeString;
        },

        // 명언 업데이트
        updateQuote() {
            const quotes = [
                "학습은 가장 좋은 투자입니다.",
                "꾸준함이 최고의 재능입니다.",
                "오늘의 노력이 내일의 성공을 만듭니다.",
                "실패는 성공의 어머니입니다.",
                "작은 진전도 큰 성취의 시작입니다."
            ];
            
            const randomQuote = quotes[Math.floor(Math.random() * quotes.length)];
            const quoteElement = document.getElementById('quote-of-day');
            if (quoteElement) quoteElement.textContent = `"${randomQuote}"`;
        },

        // D-Day 업데이트
        updateDdayCount() {
            const examDate = new Date(ACIUApp.data.userSettings.examDate);
            const today = new Date();
            const diffTime = examDate.getTime() - today.getTime();
            const diffDays = Math.ceil(diffTime / (1000 * 60 * 60 * 24));
            
            const ddayElement = document.getElementById('exam-dday');
            if (ddayElement) ddayElement.textContent = `D-${Math.max(0, diffDays)}일`;
        },

        // 설정 모달 표시
        showSettingsModal() {
            const nameInput = document.getElementById('user-name-input');
            const dateInput = document.getElementById('exam-date-input');
            const modal = document.getElementById('settings-modal');
            
            if (nameInput) nameInput.value = ACIUApp.data.userSettings.name;
            if (dateInput) dateInput.value = ACIUApp.data.userSettings.examDate;
            if (modal) modal.classList.remove('hidden');
        },

        // 설정 모달 숨기기
        hideSettingsModal() {
            const modal = document.getElementById('settings-modal');
            if (modal) modal.classList.add('hidden');
        },

        // 설정 저장
        saveSettings() {
            const nameInput = document.getElementById('user-name-input');
            const dateInput = document.getElementById('exam-date-input');
            
            const name = nameInput ? nameInput.value.trim() : '';
            const examDate = dateInput ? dateInput.value : '';
            
            if (name) {
                ACIUApp.data.userSettings.name = name;
            }
            if (examDate) {
                ACIUApp.data.userSettings.examDate = examDate;
            }
            
            ACIUApp.data.userSettings.lastUpdated = new Date().toISOString();
            ACIUApp.data.saveUserSettings();
            this.updateHeader();
            this.hideSettingsModal();
        },

        // 통계 데이터 불러오기
        loadStatisticsData() {
            console.log('통계 데이터 불러오기 시작');
            
            // 통계 섹션 표시
            const statsSection = document.getElementById('statistics-section');
            if (statsSection) statsSection.classList.remove('hidden');
            
            console.log('통계 데이터 불러오기 완료');
        },

        // 오류 메시지 표시
        showErrorMessage(message) {
            alert(message);
        },

        // 홈으로 돌아가기
        goToHome() {
            console.log('홈으로 돌아가기');
            
            // 모든 학습 화면 숨기기
            const screens = [
                'basic-learning-screen',
                'large-category-learning-screen', 
                'mid-category-learning-screen'
            ];
            
            screens.forEach(screenId => {
                const screen = document.getElementById(screenId);
                if (screen) screen.classList.add('hidden');
            });
            
            // 현재 모드 업데이트
            const modeElement = document.getElementById('current-mode');
            if (modeElement) modeElement.textContent = '모드 선택';
        },

        // 중분류 선택 관리자
        categorySelection: {
            selectedLargeCategory: null,
            selectedMidCategory: null,
            isOtherCategoryOpen: false,
            
            // 대분류 선택 시 중분류 표시
            showMidCategories(largeCategory) {
                console.log('showMidCategories 호출:', largeCategory);
                
                this.selectedLargeCategory = largeCategory;
                this.selectedMidCategory = null;
                this.isOtherCategoryOpen = false;
                
                const container = document.querySelector('.main-category-buttons');
                if (!container) {
                    console.error('main-category-buttons 컨테이너를 찾을 수 없습니다');
                    return;
                }
                
                container.innerHTML = '';
                
                // 메인 카테고리 버튼들 생성
                const mainCats = ACIUApp.data.mainCategories[largeCategory] || [];
                mainCats.forEach(cat => {
                    const button = this.createMainCategoryButton(cat.name, cat.count);
                    container.appendChild(button);
                });
                
                // 기타 버튼 생성 (기타 카테고리가 있는 경우)
                const otherCats = ACIUApp.data.otherCategories[largeCategory] || [];
                if (otherCats.length > 0) {
                    const otherCount = otherCats.reduce((sum, cat) => sum + cat.count, 0);
                    const otherButton = this.createOtherButton(otherCount);
                    container.appendChild(otherButton);
                }
                
                console.log('중분류 버튼 생성 완료');
            },
            
            // 메인 카테고리 버튼 생성
            createMainCategoryButton(name, count) {
                const button = document.createElement('button');
                button.className = 'main-category-button';
                button.innerHTML = `
                    ${name}
                    <span class="category-count">(${count}문제)</span>
                `;
                button.onclick = () => {
                    console.log('중분류 버튼 클릭:', name);
                    this.selectMidCategory(name);
                };
                return button;
            },
            
            // 기타 버튼 생성
            createOtherButton(totalCount) {
                const button = document.createElement('button');
                button.className = 'main-category-button other-button';
                button.innerHTML = `
                    기타 세부분류
                    <span class="category-count">(${totalCount}문제)</span>
                `;
                button.onclick = () => {
                    console.log('기타 버튼 클릭');
                    this.toggleOtherCategories();
                };
                return button;
            },
            
            // 중분류 선택 처리
            selectMidCategory(categoryName) {
                console.log('selectMidCategory 호출:', categoryName);
                this.selectedMidCategory = categoryName;
                this.updateBreadcrumb();
                this.startQuestionMode();
            },
            
            // 브레드크럼 업데이트
            updateBreadcrumb() {
                const largeCatElement = document.getElementById('selected-large-category');
                const midCatElement = document.getElementById('selected-mid-category');
                
                if (largeCatElement) largeCatElement.textContent = this.selectedLargeCategory || '없음';
                if (midCatElement) midCatElement.textContent = this.selectedMidCategory || '없음';
            },
            
            // 문제 모드 시작
            startQuestionMode() {
                console.log('startQuestionMode 호출');
                
                // 문제 표시 영역 표시
                const questionArea = document.getElementById('mid-category-question-area');
                if (questionArea) {
                    questionArea.classList.remove('hidden');
                    
                    // 더미 문제 표시
                    const questionText = document.getElementById('mid-category-question-text');
                    if (questionText) {
                        questionText.textContent = `${this.selectedLargeCategory} > ${this.selectedMidCategory} 관련 더미 문제입니다.`;
                    }
                    
                    // 더미 답안 버튼 생성
                    this.createDummyAnswerButtons();
                    
                    console.log('문제 표시 완료');
                } else {
                    console.error('문제 표시 영역을 찾을 수 없습니다');
                }
            },
            
            // 더미 답안 버튼 생성
            createDummyAnswerButtons() {
                const container = document.getElementById('mid-category-answer-buttons');
                if (!container) return;
                
                container.innerHTML = '';
                container.className = 'grid grid-cols-2 gap-4 mb-6';
                
                // 진위형 버튼 생성
                const oButton = document.createElement('button');
                oButton.className = 'answer-btn bg-blue-500 hover:bg-blue-600 text-white font-bold py-3 px-6 rounded-lg';
                oButton.textContent = 'O';
                oButton.onclick = () => alert('O 선택됨 (더미)');
                container.appendChild(oButton);
                
                const xButton = document.createElement('button');
                xButton.className = 'answer-btn bg-red-500 hover:bg-red-600 text-white font-bold py-3 px-6 rounded-lg';
                xButton.textContent = 'X';
                xButton.onclick = () => alert('X 선택됨 (더미)');
                container.appendChild(xButton);
            },
            
            // 기타 카테고리 토글
            toggleOtherCategories() {
                console.log('toggleOtherCategories 호출');
                // 기타 카테고리 토글 로직 (향후 구현)
            }
        }
    },

    // 학습 엔진
    learning: {
        // 학습 상태 변수
        currentQuestionIndex: 0,
        filteredQuestions: [],
        selectedAnswer: null,
        currentQuestion: null,

        // 학습 모드 시작
        startMode(mode) {
            console.log(`학습 모드 시작: ${mode}`);
            
            try {
                if (mode === 'basic') {
                    this.startBasicLearning();
                } else if (mode === 'layer1') {
                    this.startLargeCategoryLearning();
                } else if (mode === 'layer2') {
                    this.startMidCategoryLearning();
                } else {
                    alert(`${mode} 모드가 준비 중입니다.`);
                }
            } catch (error) {
                console.error('학습 모드 시작 에러:', error);
                alert('학습 모드 시작 중 에러가 발생했습니다: ' + error.message);
            }
        },

        // 기본 학습 시작
        startBasicLearning() {
            console.log('기본 학습 모드 시작');
            
            // 기본 학습 화면 표시
            const screen = document.getElementById('basic-learning-screen');
            if (screen) {
                screen.classList.remove('hidden');
                console.log('기본 학습 화면 표시됨');
            } else {
                console.error('기본 학습 화면을 찾을 수 없습니다');
            }
            
            // 현재 모드 업데이트
            const modeElement = document.getElementById('current-mode');
            if (modeElement) modeElement.textContent = '기본 학습';
        },

        // 대분류 학습 시작
        startLargeCategoryLearning() {
            console.log('대분류 학습 모드 시작');
            
            // 대분류 학습 화면 표시
            const screen = document.getElementById('large-category-learning-screen');
            if (screen) {
                screen.classList.remove('hidden');
                console.log('대분류 학습 화면 표시됨');
            } else {
                console.error('대분류 학습 화면을 찾을 수 없습니다');
            }
            
            // 현재 모드 업데이트
            const modeElement = document.getElementById('current-mode');
            if (modeElement) modeElement.textContent = '대분류 학습';
        },

        // 중분류 학습 시작
        startMidCategoryLearning() {
            console.log('중분류 학습 모드 시작');
            
            // 중분류 학습 화면 표시
            const screen = document.getElementById('mid-category-learning-screen');
            if (screen) {
                screen.classList.remove('hidden');
                console.log('중분류 학습 화면 표시됨');
            } else {
                console.error('중분류 학습 화면을 찾을 수 없습니다');
                return;
            }
            
            // 현재 모드 업데이트
            const modeElement = document.getElementById('current-mode');
            if (modeElement) modeElement.textContent = '중분류 학습';
            
            // 대분류 버튼들 생성
            this.createLargeCategoryButtons();
        },

        // 대분류 버튼들 생성
        createLargeCategoryButtons() {
            console.log('대분류 버튼 생성 시작');
            
            const container = document.querySelector('.main-category-buttons');
            if (!container) {
                console.error('main-category-buttons 컨테이너를 찾을 수 없습니다');
                return;
            }
            
            container.innerHTML = '';
            
            const largeCategories = ['06재산보험', '07특종보험', '08배상책임보험', '09해상보험'];
            
            largeCategories.forEach(category => {
                const button = document.createElement('button');
                button.className = 'main-category-button';
                button.innerHTML = `
                    ${category}
                    <span class="category-count">(${this.getTotalQuestionCount(category)}문제)</span>
                `;
                button.onclick = () => {
                    console.log('대분류 버튼 클릭:', category);
                    ACIUApp.ui.categorySelection.showMidCategories(category);
                };
                container.appendChild(button);
            });
            
            console.log('대분류 버튼 생성 완료');
        },

        // 대분류별 총 문제 수 계산
        getTotalQuestionCount(largeCategory) {
            const mainCats = ACIUApp.data.mainCategories[largeCategory] || [];
            const otherCats = ACIUApp.data.otherCategories[largeCategory] || [];
            
            const mainCount = mainCats.reduce((sum, cat) => sum + cat.count, 0);
            const otherCount = otherCats.reduce((sum, cat) => sum + cat.count, 0);
            
            return mainCount + otherCount;
        }
    }
};

// ACIUApp 초기화 함수
ACIUApp.init = async function() {
    console.log('ACIUApp 초기화 시작');
    
    try {
        // 브라우저 호환성 확인
        this.compatibility.checkBrowserSupport();
        
        // 데이터 로딩
        await this.data.loadQuestions();
        this.data.loadUserSettings();
        this.data.loadLearningHistory();
        
        // UI 초기화
        this.ui.init();
        
        // 타이머 설정
        this.timers.timeUpdate = setInterval(() => this.ui.updateCurrentTime(), 1000);
        this.timers.quoteUpdate = setInterval(() => this.ui.updateQuote(), 300000); // 5분마다
        this.timers.ddayUpdate = setInterval(() => this.ui.updateDdayCount(), 3600000); // 1시간마다
        
        console.log('ACIUApp 초기화 완료');
        
    } catch (error) {
        console.error('ACIUApp 초기화 실패:', error);
        alert('앱 초기화 중 오류가 발생했습니다: ' + error.message);
    }
};

// 테스트 함수들
window.testMidCategoryFunction = function() {
    console.log('=== 중분류 학습 테스트 시작 ===');
    console.log('ACIUApp 존재 여부:', typeof ACIUApp !== 'undefined');
    console.log('ACIUApp.learning 존재 여부:', typeof ACIUApp.learning !== 'undefined');
    console.log('ACIUApp.learning.startMode 존재 여부:', typeof ACIUApp.learning.startMode !== 'undefined');
    console.log('mid-category-learning-screen 요소 존재 여부:', !!document.getElementById('mid-category-learning-screen'));
    console.log('=== 테스트 완료 ===');
};

window.testMidCategoryButton = function() {
    console.log('=== 중분류 학습 버튼 클릭 테스트 ===');
    
    try {
        // 1. ACIUApp 존재 확인
        if (typeof ACIUApp === 'undefined') {
            console.error('❌ ACIUApp이 정의되지 않음');
            alert('ACIUApp이 로드되지 않았습니다.');
            return;
        }
        console.log('✅ ACIUApp 존재 확인');
        
        // 2. learning 객체 확인
        if (typeof ACIUApp.learning === 'undefined') {
            console.error('❌ ACIUApp.learning이 정의되지 않음');
            alert('학습 모듈이 로드되지 않았습니다.');
            return;
        }
        console.log('✅ ACIUApp.learning 존재 확인');
        
        // 3. startMode 함수 확인
        if (typeof ACIUApp.learning.startMode !== 'function') {
            console.error('❌ startMode 함수가 정의되지 않음');
            alert('학습 모드 시작 함수가 없습니다.');
            return;
        }
        console.log('✅ startMode 함수 존재 확인');
        
        // 4. 중분류 학습 화면 요소 확인
        const midCategoryScreen = document.getElementById('mid-category-learning-screen');
        if (!midCategoryScreen) {
            console.error('❌ 중분류 학습 화면 요소가 없음');
            alert('중분류 학습 화면을 찾을 수 없습니다.');
            return;
        }
        console.log('✅ 중분류 학습 화면 요소 존재 확인');
        
        // 5. 실제 함수 호출
        console.log('🔄 startMode("layer2") 호출 시도...');
        ACIUApp.learning.startMode('layer2');
        console.log('✅ startMode("layer2") 호출 완료');
        
    } catch (error) {
        console.error('❌ 에러 발생:', error);
        alert('중분류 학습 시작 중 에러가 발생했습니다: ' + error.message);
    }
};

// 페이지 로드 시 초기화
window.addEventListener('load', function() {
    ACIUApp.init();
});