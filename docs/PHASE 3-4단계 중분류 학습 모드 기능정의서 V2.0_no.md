📋 PHASE 3-4단계 중분류 학습 모드 기능정의서 V2.0 (노팀장 수정안) - 조대표님 지시 반영 (2025.07.27 일요일 KST)
작성자: 노팀장 (기술자문) - 코코치 초안 수정
최종 승인: 조대표님 확정
목표: AICU Quiz App의 중분류 학습 모드 UI 설계 완성 (12개 중분류 버튼 최적화 배치)

## 1. 조대표님 결정사항 반영
- **중분류 학습은 시험공부에 필수적이므로 반드시 구현**
- **현재는 UI 확정에 집중** (더미 데이터 활용)
- **UI 완성 후 실제 문제 데이터로 검증 진행**
- **4개 대분류 × 3-4개 중분류 = 약 12개 중분류 버튼**

## 2. 핵심 UI 설계 원칙 (노팀장 기술자문)

### 2.1 단계적 선택 구조
```
1단계: 대분류 4개 버튼 (한 줄 배치)
   ↓
2단계: 선택된 대분류의 중분류 3-4개 동적 표시
   ↓
3단계: 문제 풀이 (기존 로직 재사용)
```

### 2.2 UI 최적화 전략
- **모바일 우선**: 12개 버튼이 한번에 노출되지 않도록 단계적 표시
- **명확한 선택 상태**: 활성화된 버튼 시각적 강조 (색상, 굵기)
- **경로 표시**: "[대분류] > [중분류] > 문제 X/Y" 브레드크럼
- **반응형 설계**: 데스크톱 4개/모바일 2개씩 배치

### 2.3 메모리 효율성
- 선택되지 않은 중분류 데이터는 메모리에서 제외
- 필요시에만 중분류 목록 동적 생성
- 이전 선택 기록은 LocalStorage에 간단히 저장

## 3. 중분류 학습 페이지 UI 구성 (수정안)

### 3.1 화면 레이아웃
```
┌─────────────────────────────────────┐
│ 🏠 홈으로                           │
├─────────────────────────────────────┤
│ 오늘의 명언 (기존 헤더 유지)         │
├─────────────────────────────────────┤
│ 보유 문제 현황 (statistics1.html 스타일) │
│ 📊 인스교재: 789문제 (선택형 389, 진위형 400) │
│ 📊 중개사시험: 590문제 (선택형 390, 진위형 200) │
│ 📊 총계: 1,379문제                   │
├─────────────────────────────────────┤
│ 학습 진도 현황 (중분류별 상세)       │
│ 🎯 4과목 평균: 15% (합격 커트라인: 60%) │
│ 📈 과목별 커트라인: 40점 이상        │
├─────────────────────────────────────┤
│ 1단계: 대분류 선택                   │
│ [재산보험] [특종보험] [배상책임] [해상] │
├─────────────────────────────────────┤
│ 2단계: 중분류 선택 (동적 표시)       │
│ [화재보험] [자동차손해] [기타손해]    │
├─────────────────────────────────────┤
│ 선택된 경로: 재산보험 > 화재보험      │
├─────────────────────────────────────┤
│ 문제 표시 영역                       │
│ (기존 문제 풀이 UI 재사용)           │
└─────────────────────────────────────┘
```

### 3.2 버튼 배치 세부 설계

#### 대분류 버튼 (4개)
```css
/* 데스크톱: 한 줄 배치 */
.large-category-buttons {
    display: grid;
    grid-template-columns: repeat(4, 1fr);
    gap: 10px;
}

/* 모바일: 2줄 배치 */
@media (max-width: 768px) {
    .large-category-buttons {
        grid-template-columns: repeat(2, 1fr);
    }
}
```

#### 중분류 버튼 (동적 3-4개)
```css
/* 최대 4개까지 한 줄 배치 */
.mid-category-buttons {
    display: flex;
    flex-wrap: wrap;
    justify-content: center;
    gap: 8px;
    margin-top: 15px;
}

/* 모바일에서는 2개씩 배치, 터치 최적화 */
@media (max-width: 768px) {
    .mid-category-buttons button {
        flex-basis: calc(50% - 4px);
        min-height: 44px; /* 터치 인터페이스 최소 크기 */
        font-size: 14px;
    }
}
```

### 3.3 상태 관리 로직 (서대리 의견 반영)
```javascript
ACIUApp.ui.categorySelection = {
    selectedLargeCategory: null,
    selectedMidCategory: null,
    
    // 대분류 선택 시
    selectLargeCategory(category) {
        this.selectedLargeCategory = category;
        this.selectedMidCategory = null; // 중분류 초기화
        this.showMidCategories(category);
        this.updateBreadcrumb();
    },
    
    // 중분류 선택 시
    selectMidCategory(category) {
        this.selectedMidCategory = category;
        this.startQuestionMode();
        this.updateBreadcrumb();
    },
    
    // 경로 표시 업데이트 (진행률 포함 - 간소화)
    updateBreadcrumb() {
        let path = this.selectedLargeCategory;
        if (this.selectedMidCategory) {
            path += ' > ' + this.selectedMidCategory;
            // 문제 진행률 표시 (간소화된 형태)
            if (ACIUApp.learning.currentQuestionIndex !== undefined) {
                const current = ACIUApp.learning.currentQuestionIndex + 1;
                const total = ACIUApp.learning.filteredQuestions.length;
                const percentage = Math.round((current / total) * 100);
                path += ` > ${current}/${total} (${percentage}%)`;
            }
        }
        document.getElementById('category-path').textContent = path;
    },
    
    // LocalStorage 상태 저장/복원
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
                this.updateBreadcrumb();
            }
        } catch (e) {
            console.log('상태 복원 실패:', e);
        }
    }
};
```

## 4. 더미 데이터 구조 (실제 CSV 분석 후 확정 필요)

### 4.1 대분류별 중분류 매핑 (주의: ins_master_db.csv 검증 필요)
```javascript
// 주의: 실제 CSV의 LAYER2 값 분석 후 정확한 매핑으로 업데이트 필요
ACIUApp.data.categoryMapping = {
    '재산보험': ['화재보험', '자동차손해보험', '기타손해보험'],
    '특종보험': ['상해보험', '질병보험', '간병보험', '치아보험'],
    '배상책임보험': ['일반배상책임', '생산물배상책임', '전문직배상책임'],
    '해상보험': ['적하보험', '선박보험', '해상운송보험']
};

// 에러 처리: 포괄적 에러 핸들링
ACIUApp.ui.handleError = function(errorType, details) {
    const errorMessages = {
        'category_empty': `"${details}" 카테고리에는 문제가 없습니다.`,
        'loading_failed': '데이터 로딩에 실패했습니다. 다시 시도해주세요.',
        'network_error': '네트워크 연결을 확인해주세요.'
    };
    
    document.getElementById('error-message').textContent = errorMessages[errorType];
    document.getElementById('error-message').style.display = 'block';
    
    // 해당 버튼 비활성화 (category_empty인 경우)
    if (errorType === 'category_empty') {
        const button = document.querySelector(`[data-category="${details}"]`);
        if (button) {
            button.disabled = true;
            button.classList.add('disabled');
        }
    }
};

// 로딩 상태 표시
ACIUApp.ui.showLoading = function(message = "데이터를 불러오는 중입니다...") {
    document.getElementById('loading-indicator').textContent = message;
    document.getElementById('loading-indicator').style.display = 'block';
};

ACIUApp.ui.hideLoading = function() {
    document.getElementById('loading-indicator').style.display = 'none';
};
```

## 5. 구현 우선순위 (노팀장 기술자문)

### Phase 1: UI 뼈대 구성 (당일 완료)
1. **ins_master_db.csv LAYER2 값 사전 검증** (서대리 의견 반영)
2. HTML 구조 배치
3. CSS 스타일링 (반응형 + 터치 최적화)
4. 버튼 클릭 이벤트 연결
5. 선택 상태 시각화
6. **브레드크럼 진행률 표시 (간소화)**
7. **로딩 인디케이터 구현**

### Phase 2: 더미 데이터 연동 (다음일)
1. 대분류 선택 → 중분류 표시
2. 중분류 선택 → 문제 모드 진입
3. 브레드크럼 네비게이션
4. **포괄적 에러 처리 및 로딩 메시지**
5. **LocalStorage 상태 복원 기능**

### Phase 3: 실제 데이터 연동 (UI 확정 후)
1. **검증된 LAYER2 값으로 실제 카테고리 매핑**
2. 문제 필터링 로직 구현
3. 통계 계산 연동

## 6. 코코치 임무 지시사항

조대표님, 다음 임무를 코코치에게 지시해 주십시오:

### 6.1 UI/UX 세부 설계 (중요도 높음)
- 12개 중분류 버튼의 최적 배치 방안 구체화
- 모바일 환경에서의 사용성 검토 (44px 최소 터치 영역)
- 버튼 크기, 간격, 색상 팔레트 제안
- **접근성 고려**: WCAG 2.1 AA 기준 색상 대비(4.5:1), 키보드 네비게이션, ARIA 라벨

### 6.2 사용자 경험 시나리오 작성 (중요도 높음)
- 대분류 → 중분류 선택 흐름 시나리오
- **오류 상황 대응**: 카테고리 없음, 문제 없음, 로딩 실패, 네트워크 오류
- **복귀 시나리오**: 홈 → 재진입 시 이전 선택 상태 복원
- **학습 연속성**: 다른 모드에서 돌아왔을 때 진행 상황 유지

### 6.3 성능 최적화 방안 (중요도 중간)
- 중분류 버튼 동적 생성 시 성능 고려
- 메모리 사용량 최소화 전략
- 로딩 시간 단축 방안
- **오프라인 대응**: 네트워크 오류 시 캐시된 데이터 활용

## 7. 다음 단계 및 우선순위 (코코치 의견 반영)
1. **조대표님 최종 승인** → 본 기능정의서 확정
2. **노팀장/서대리 CSV 검증** → ins_master_db.csv의 실제 LAYER2 값 분석
3. **코코치 UI/UX 세부 설계** → 상세 UI 가이드 작성
4. **서대리 구현** → index3.5.html 개발
5. **실제 데이터 검증** → CSV 데이터 연동 테스트

### 향후 우선순위 결정 대기 중:
- 실제 데이터 연동 vs 진위형/선택형 필터링 vs 이어서 풀기 기능
- 조대표님의 최종 학습 목표에 따른 우선순위 결정 필요

---
*작성: 노팀장 (기술자문) | 수정일: 2025.07.27 | 상태: 조대표님 승인 대기*