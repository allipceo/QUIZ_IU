📋 PHASE 3-4단계 중분류학습 모드 작업지시서 V1.0 (2025.07.27 일요일 KST)
작성자: 노팀장 (기술자문)
승인자: 조대표님 (최종 승인 완료)
목표: AICU Quiz App의 중분류 학습 모드 구현 (index3.5.html)

## 1. 작업 개요

### 1.1 프로젝트 상황
- **현재 상태**: index3.4.html (대분류 학습 모드) 완료
- **목표 파일**: index3.5.html (중분류 학습 모드) 신규 개발
- **기반 구조**: 기존 ACIUApp 네임스페이스 패턴 유지

### 1.2 핵심 구현 목표
- 2단계 선택 구조: 대분류 → 중분류 → 문제 풀이
- 12개 중분류 버튼의 최적 배치
- 모바일 반응형 설계 및 접근성 확보
- 실제 CSV 데이터 기반 구현

## 2. 팀별 역할 및 책임

### 2.1 서대리 (메인 개발자)
- index3.4.html → index3.5.html 복사 후 중분류 모드 구현
- HTML/CSS/JavaScript 코딩 전담
- ACIUApp 네임스페이스 패턴 준수
- 모든 기능의 실제 구현 및 테스트

### 2.2 노팀장 (기술자문)
- ins_master_db.csv의 LAYER2 값 사전 분석 및 검증
- 데이터 구조 및 필터링 로직 자문
- 서대리 코드 검토 및 기술적 이슈 해결 지원
- 성능 최적화 방안 제시

### 2.3 코코치 (UI/UX 설계)
- 12개 중분류 버튼 배치 세부 설계
- 모바일 사용성 및 접근성 가이드라인 제공
- 브레드크럼 및 진행률 표시 디자인
- 에러 메시지 및 로딩 상태 UI 설계

## 3. Phase 1: 사전 분석 및 UI 뼈대 구성 (당일 완료)

### 3.1 노팀장 임무: CSV 데이터 분석 (최우선 완료)
```bash
# 작업 목표: ins_master_db.csv의 LAYER2 값 분석 및 즉시 공유
1. Phase 1 시작 전 CSV 분석 완료
2. LAYER1별 LAYER2 값 추출 및 검증
3. 각 중분류별 문제 수 집계
4. 더미 데이터 구조 확정 후 서대리/코코치에게 즉시 전달
```

**분석 항목 (자동차보험 제외):**
- 재산보험의 중분류 목록 및 문제 수
- 특종보험의 중분류 목록 및 문제 수  
- 배상책임보험의 중분류 목록 및 문제 수
- 해상보험의 중분류 목록 및 문제 수

**주의사항**: 자동차보험은 전문영역에 없으므로 제외

### 3.2 서대리 임무: 파일 준비 및 HTML 구조
```html
<!-- 작업 목표: index3.5.html 기본 구조 완성 -->
1. index3.4.html → index3.5.html 복사
2. 중분류 선택 UI 추가
3. 브레드크럼 네비게이션 영역 추가
4. 에러 메시지 및 로딩 인디케이터 영역 추가
```

**필수 HTML 요소:**
```html
<!-- 대분류 선택 영역 -->
<div id="large-category-selection" class="category-section">
    <h3>대분류 선택</h3>
    <div class="large-category-buttons">
        <!-- 동적 생성 -->
    </div>
</div>

<!-- 중분류 선택 영역 -->
<div id="mid-category-selection" class="category-section" style="display:none;">
    <h3>중분류 선택</h3>
    <div class="mid-category-buttons">
        <!-- 동적 생성 -->
    </div>
</div>

<!-- 선택된 경로 표시 -->
<div id="category-path-display" class="breadcrumb">
    <span id="category-path"></span>
</div>

<!-- 에러 메시지 -->
<div id="error-message" class="error-display" style="display:none;"></div>

<!-- 로딩 인디케이터 -->
<div id="loading-indicator" class="loading-display" style="display:none;"></div>
```

### 3.3 CSS 스타일링 (반응형 + 터치 최적화)
```css
/* 서대리 구현 목표 */
.large-category-buttons {
    display: grid;
    grid-template-columns: repeat(4, 1fr);
    gap: 10px;
    margin-bottom: 20px;
}

.mid-category-buttons {
    display: flex;
    flex-wrap: wrap;
    justify-content: center;
    gap: 8px;
    margin-top: 15px;
}

.category-button {
    min-height: 44px; /* 터치 최적화 */
    padding: 12px 16px;
    border: 2px solid #ddd;
    border-radius: 8px;
    background: white;
    cursor: pointer;
    transition: all 0.3s ease;
}

.category-button:hover {
    background: #f0f9ff;
    border-color: #3b82f6;
}

.category-button.active {
    background: #3b82f6;
    color: white;
    border-color: #2563eb;
    font-weight: bold;
}

.category-button:disabled {
    background: #f3f4f6;
    color: #9ca3af;
    cursor: not-allowed;
}

/* 모바일 반응형 */
@media (max-width: 768px) {
    .large-category-buttons {
        grid-template-columns: repeat(2, 1fr);
    }
    
    .mid-category-buttons button {
        flex-basis: calc(50% - 4px);
        min-height: 44px;
        font-size: 14px;
    }
}

/* 접근성 고려 */
.category-button:focus {
    outline: 3px solid #fbbf24;
    outline-offset: 2px;
}

/* 브레드크럼 */
.breadcrumb {
    background: #f8fafc;
    padding: 12px 16px;
    border-radius: 6px;
    margin: 16px 0;
    font-weight: 500;
    color: #374151;
}

/* 에러 및 로딩 */
.error-display {
    background: #fef2f2;
    color: #dc2626;
    padding: 12px 16px;
    border-radius: 6px;
    border: 1px solid #fecaca;
}

.loading-display {
    background: #eff6ff;
    color: #2563eb;
    padding: 12px 16px;
    border-radius: 6px;
    text-align: center;
}
```

## 4. Phase 2: JavaScript 기능 구현 (다음일)

### 4.1 데이터 구조 정의 (노팀장 분석 결과 반영)
```javascript
// 작업 목표: 중분류 선택 및 문제 연동 완성

// 1. 데이터 구조 정의 (노팀장 분석 결과 대기 중)
// 주의: 자동차보험은 전문영역에 없으므로 제외
ACIUApp.data.categoryMapping = {
    // 노팀장 CSV 분석 완료 후 정확한 매핑으로 업데이트
    // 예시 구조 (실제 데이터로 교체 필요):
    '재산보험': ['화재보험', '기타재산보험'], // 자동차손해보험 제외
    '특종보험': ['상해보험', '질병보험', '간병보험'],
    '배상책임보험': ['일반배상책임', '생산물배상책임'],
    '해상보험': ['적하보험', '선박보험']
};

// 2. 상태 복원 정책: 대분류만 복원 (조대표님 지시)
ACIUApp.ui.categorySelection = {
    restoreState() {
        try {
            const state = JSON.parse(localStorage.getItem('aciu_category_state'));
            if (state && state.largeCategory) {
                // 대분류만 복원, 중분류는 초기화
                this.selectedLargeCategory = state.largeCategory;
                this.selectedMidCategory = null;
                this.showMidCategories(state.largeCategory);
                this.updateBreadcrumb();
            }
        } catch (e) {
            console.log('상태 복원 실패:', e);
        }
    }
};

// 2. 카테고리 선택 관리자
ACIUApp.ui.categorySelection = {
    selectedLargeCategory: null,
    selectedMidCategory: null,
    
    // 대분류 선택 처리
    selectLargeCategory(category) {
        this.selectedLargeCategory = category;
        this.selectedMidCategory = null;
        this.showMidCategories(category);
        this.updateBreadcrumb();
        this.saveState();
    },
    
    // 중분류 선택 처리
    selectMidCategory(category) {
        this.selectedMidCategory = category;
        this.startQuestionMode();
        this.updateBreadcrumb();
        this.saveState();
    },
    
    // 중분류 버튼 동적 생성
    showMidCategories(largeCategory) {
        const midCategories = ACIUApp.data.categoryMapping[largeCategory];
        const container = document.querySelector('.mid-category-buttons');
        
        // 이전 버튼들 제거
        container.innerHTML = '';
        
        // 새 버튼들 생성
        midCategories.forEach(midCategory => {
            const button = this.createCategoryButton(midCategory, 'mid');
            container.appendChild(button);
        });
        
        // 중분류 영역 표시
        document.getElementById('mid-category-selection').style.display = 'block';
    },
    
    // 브레드크럼 업데이트 (간소화된 형태)
    updateBreadcrumb() {
        let path = this.selectedLargeCategory || '';
        if (this.selectedMidCategory) {
            path += ' > ' + this.selectedMidCategory;
            
            // 문제 진행률 (간소화)
            if (ACIUApp.learning.currentQuestionIndex !== undefined) {
                const current = ACIUApp.learning.currentQuestionIndex + 1;
                const total = ACIUApp.learning.filteredQuestions.length;
                const percentage = Math.round((current / total) * 100);
                path += ` > ${current}/${total} (${percentage}%)`;
            }
        }
        document.getElementById('category-path').textContent = path;
    },
    
    // 상태 저장/복원
    saveState() {
        const state = {
            largeCategory: this.selectedLargeCategory,
            midCategory: this.selectedMidCategory,
            questionIndex: ACIUApp.learning.currentQuestionIndex || 0
        };
        localStorage.setItem('aciu_category_state', JSON.stringify(state));
    },
    
    restoreState() {
        try {
            const state = JSON.parse(localStorage.getItem('aciu_category_state'));
            if (state) {
                this.selectedLargeCategory = state.largeCategory;
                this.selectedMidCategory = state.midCategory;
                ACIUApp.learning.currentQuestionIndex = state.questionIndex;
                
                if (this.selectedLargeCategory) {
                    this.showMidCategories(this.selectedLargeCategory);
                    this.updateBreadcrumb();
                }
            }
        } catch (e) {
            console.log('상태 복원 실패:', e);
        }
    }
};

// 3. 에러 처리
ACIUApp.ui.handleError = function(errorType, details) {
    const errorMessages = {
        'category_empty': `"${details}" 카테고리에는 문제가 없습니다.`,
        'loading_failed': '데이터 로딩에 실패했습니다. 다시 시도해주세요.',
        'network_error': '네트워크 연결을 확인해주세요.'
    };
    
    document.getElementById('error-message').textContent = errorMessages[errorType];
    document.getElementById('error-message').style.display = 'block';
    
    if (errorType === 'category_empty') {
        const button = document.querySelector(`[data-category="${details}"]`);
        if (button) {
            button.disabled = true;
            button.classList.add('disabled');
        }
    }
};

// 4. 로딩 상태 관리
ACIUApp.ui.showLoading = function(message = "데이터를 불러오는 중입니다...") {
    document.getElementById('loading-indicator').textContent = message;
    document.getElementById('loading-indicator').style.display = 'block';
};

ACIUApp.ui.hideLoading = function() {
    document.getElementById('loading-indicator').style.display = 'none';
};
```

### 4.2 문제 필터링 및 학습 모드 연동
```javascript
// 서대리 구현 목표: 중분류 선택 후 문제 표시

ACIUApp.learning.startMidCategoryMode = function(largeCategory, midCategory) {
    // 로딩 표시
    ACIUApp.ui.showLoading('문제를 불러오는 중입니다...');
    
    try {
        // 문제 필터링 (LAYER1 + LAYER2 기준)
        this.filteredQuestions = ACIUApp.data.questions.filter(q => 
            q.LAYER1 === largeCategory && q.LAYER2 === midCategory
        );
        
        // 문제가 없는 경우 처리
        if (this.filteredQuestions.length === 0) {
            ACIUApp.ui.handleError('category_empty', `${largeCategory} > ${midCategory}`);
            return;
        }
        
        // 문제 풀이 시작
        this.currentQuestionIndex = 0;
        this.displayQuestion();
        this.updateStats();
        
        // 화면 전환
        ACIUApp.ui.hideAllScreens();
        document.getElementById('common-learning-screen').style.display = 'block';
        
    } catch (error) {
        ACIUApp.ui.handleError('loading_failed', null);
        console.error('중분류 모드 시작 오류:', error);
    } finally {
        ACIUApp.ui.hideLoading();
    }
};
```

## 5. Phase 3: 실제 데이터 연동 및 테스트 (UI 확정 후)

### 5.1 노팀장 임무: 데이터 검증 및 최적화
- 실제 필터링 로직 성능 테스트
- 메모리 사용량 최적화 검토
- 에러 케이스 추가 발견 및 대응

### 5.2 서대리 임무: 통합 테스트 및 디버깅
- 전체 기능 통합 테스트
- 브라우저 호환성 확인
- 모바일 환경 테스트
- LocalStorage 동작 검증

## 6. 품질 관리 및 검수 기준

### 6.1 기능 요구사항 체크리스트
- [ ] 대분류 4개 버튼이 정상 표시되는가?
- [ ] 대분류 선택 시 중분류가 동적으로 생성되는가?
- [ ] 중분류 선택 시 해당 문제들이 필터링되는가?
- [ ] 브레드크럼 경로가 정확히 표시되는가?
- [ ] 문제 진행률이 올바르게 계산되는가?
- [ ] 홈 버튼으로 대문 복귀가 가능한가?
- [ ] 상태 저장/복원이 정상 작동하는가?

### 6.2 UI/UX 요구사항 체크리스트
- [ ] 모바일에서 버튼이 44px 이상인가?
- [ ] 데스크톱 4개/모바일 2개씩 배치되는가?
- [ ] 선택된 버튼이 시각적으로 강조되는가?
- [ ] 키보드 네비게이션이 가능한가?
- [ ] 에러 메시지가 명확히 표시되는가?
- [ ] 로딩 상태가 적절히 표시되는가?

### 6.3 접근성 요구사항 체크리스트
- [ ] 색상 대비가 WCAG 2.1 AA 기준을 만족하는가?
- [ ] ARIA 라벨이 적절히 설정되었는가?
- [ ] 스크린 리더로 탐색이 가능한가?
- [ ] 키보드만으로 모든 기능에 접근 가능한가?

## 7. 일정 및 마일스톤 (수정)

### 7.1 사전 준비 (즉시)
- **노팀장**: ins_master_db.csv 분석 완료 및 결과 공유
- **코코치**: '문제 없는 카테고리' UI/UX 시나리오 상세 정의

### 7.2 Day 1 (당일)
- **오전**: 서대리 파일 준비 및 HTML 구조 (정확한 데이터 기반)
- **오후**: CSS 스타일링 (statistics1.html 포맷 정확히 적용)
- **저녁**: Phase 1 완료 및 중간 검토

### 7.3 Day 2 (다음일)
- **오전**: JavaScript 로직 구현 (대분류만 상태 복원)
- **오후**: 기능 테스트 및 디버깅
- **저녁**: Phase 2 완료 및 최종 검토

### 7.3 Day 3 (최종일)
- **오전**: 실제 데이터 연동 및 통합 테스트
- **오후**: 품질 검수 및 최종 확인
- **저녁**: Phase 3 완료 및 조대표님 검수

## 8. 위험 요소 및 대응 방안

### 8.1 기술적 위험
- **CSV 데이터 불일치**: 노팀장 사전 분석으로 예방
- **성능 저하**: 필터링 로직 최적화로 대응
- **브라우저 호환성**: 폴리필 적용으로 해결

### 8.2 일정 위험
- **구현 지연**: 기능 우선순위 조정으로 대응
- **버그 발생**: 단계별 테스트로 조기 발견
- **요구사항 변경**: 조대표님과 즉시 소통

## 9. 성공 기준

### 9.1 최소 성공 기준 (Must Have)
- 2단계 카테고리 선택이 정상 작동
- 필터링된 문제가 올바르게 표시
- 모바일 환경에서 사용 가능
- 기본적인 에러 처리 구현

### 9.2 완전 성공 기준 (Should Have)
- 모든 접근성 요구사항 충족
- 상태 저장/복원 기능 완벽 작동
- 성능 최적화 완료
- 조대표님 만족도 90% 이상

---
**작업 시작 전 확인사항:**
1. 모든 팀원이 역할과 책임을 명확히 이해했는가?
2. 필요한 개발 환경과 도구가 준비되었는가?
3. 조대표님과의 소통 채널이 확보되었는가?

**조대표님, 이 작업지시서에 따라 중분류 학습 모드 개발을 시작하겠습니다!**