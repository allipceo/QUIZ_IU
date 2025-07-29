# 옵션1 구현 가이드 - 중분류 학습 모드 (2025.07.27 01:50 KST)

## 1. 확정된 데이터 구조

### 1.1 메인 카테고리 (50문제 이상)
```javascript
ACIUApp.data.mainCategories = {
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
    '08배상책임보험': [
        // 50문제 이상 없음 - "기타"로만 표시
    ],
    '09해상보험': [
        { name: '미확인', count: 165 },
        { name: '해상보험조건과보상범위', count: 57 },
        { name: '해상보험의손해사정', count: 53 }
    ]
};
```

### 1.2 기타 카테고리 (50문제 미만)
```javascript
ACIUApp.data.otherCategories = {
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
};
```

## 2. UI 레이아웃 확정

### 2.1 중분류 선택 영역
```
┌─────────────────────────────────────┐
│ 2단계: 중분류 선택                   │
├─────────────────────────────────────┤
│ [화재보험]    [재물보험재보험]       │
│ (188문제)     (130문제)             │
│                                     │
│ [패키지보험]  [기타 재산보험]        │
│ (53문제)      (28문제)              │
└─────────────────────────────────────┘

┌─────────────────────────────────────┐
│ 기타 재산보험 세부 분류 (토글)       │
├─────────────────────────────────────┤
│ [동산종합] [기업휴지] [재물보험]     │
│  (15문제)   (8문제)   (5문제)       │
└─────────────────────────────────────┘
```

### 2.2 각 대분류별 버튼 구성
- **06재산보험**: 메인 3개 + 기타 1개 = 총 4개 버튼
- **07특종보험**: 메인 3개 + 기타 1개 = 총 4개 버튼  
- **08배상책임보험**: 기타 1개만 = 총 1개 버튼
- **09해상보험**: 메인 3개 + 기타 1개 = 총 4개 버튼

## 3. 서대리 작업 지시사항

### 3.1 HTML 구조
```html
<!-- 중분류 선택 영역 -->
<div id="mid-category-selection" class="category-section" style="display:none;">
    <h3>중분류 선택</h3>
    
    <!-- 메인 카테고리 버튼들 -->
    <div class="main-category-buttons">
        <!-- 동적 생성: 메인 카테고리 + 기타 버튼 -->
    </div>
    
    <!-- 기타 카테고리 상세 (토글 방식) -->
    <div id="other-categories-detail" class="other-category-section" style="display:none;">
        <h4>기타 세부 분류</h4>
        <div class="other-category-buttons">
            <!-- 동적 생성: 기타 세부 카테고리들 -->
        </div>
        <button class="close-other-btn">접기</button>
    </div>
</div>
```

### 3.2 CSS 스타일링
```css
/* 메인 카테고리 버튼 영역 */
.main-category-buttons {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(140px, 1fr));
    gap: 12px;
    max-width: 600px;
    margin: 0 auto 20px;
}

.main-category-button {
    min-height: 60px;
    padding: 12px 16px;
    border: 2px solid #e5e7eb;
    border-radius: 8px;
    background: white;
    cursor: pointer;
    transition: all 0.3s ease;
    text-align: center;
}

.main-category-button:hover {
    background: #f0f9ff;
    border-color: #3b82f6;
    transform: translateY(-2px);
}

.main-category-button.active {
    background: #3b82f6;
    color: white;
    border-color: #2563eb;
    font-weight: bold;
}

.main-category-button.other-button {
    background: #f3f4f6;
    border-style: dashed;
}

.main-category-button.other-button:hover {
    background: #e5e7eb;
}

/* 문제 수 표시 */
.category-count {
    display: block;
    font-size: 12px;
    color: #6b7280;
    margin-top: 4px;
}

.active .category-count {
    color: #e5e7eb;
}

/* 기타 카테고리 상세 영역 */
.other-category-section {
    background: #f9fafb;
    border: 1px solid #e5e7eb;
    border-radius: 8px;
    padding: 16px;
    margin-top: 16px;
}

.other-category-buttons {
    display: flex;
    flex-wrap: wrap;
    gap: 8px;
    margin: 12px 0;
}

.other-category-button {
    font-size: 12px;
    padding: 8px 12px;
    min-height: 36px;
    border: 1px solid #d1d5db;
    border-radius: 6px;
    background: white;
    cursor: pointer;
    transition: all 0.2s ease;
}

.other-category-button:hover {
    background: #f3f4f6;
    border-color: #9ca3af;
}

.other-category-button.active {
    background: #1f2937;
    color: white;
    border-color: #374151;
}

.close-other-btn {
    width: 100%;
    padding: 8px;
    background: #e5e7eb;
    border: none;
    border-radius: 4px;
    cursor: pointer;
    margin-top: 8px;
}

/* 모바일 반응형 */
@media (max-width: 768px) {
    .main-category-buttons {
        grid-template-columns: repeat(2, 1fr);
        gap: 8px;
    }
    
    .main-category-button {
        min-height: 50px;
        font-size: 14px;
        padding: 10px 12px;
    }
    
    .other-category-button {
        font-size: 11px;
        padding: 6px 10px;
        min-height: 32px;
    }
}
```

### 3.3 JavaScript 로직
```javascript
// 중분류 선택 관리자 (수정된 버전)
ACIUApp.ui.categorySelection = {
    selectedLargeCategory: null,
    selectedMidCategory: null,
    isOtherCategoryOpen: false,
    
    // 대분류 선택 시 중분류 표시
    showMidCategories(largeCategory) {
        this.selectedLargeCategory = largeCategory;
        this.selectedMidCategory = null;
        this.isOtherCategoryOpen = false;
        
        const container = document.querySelector('.main-category-buttons');
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
        
        // 중분류 영역 표시
        document.getElementById('mid-category-selection').style.display = 'block';
        
        // 기타 카테고리 상세는 숨김
        document.getElementById('other-categories-detail').style.display = 'none';
    },
    
    // 메인 카테고리 버튼 생성
    createMainCategoryButton(name, count) {
        const button = document.createElement('button');
        button.className = 'main-category-button';
        button.innerHTML = `
            ${name}
            <span class="category-count">(${count}문제)</span>
        `;
        button.onclick = () => this.selectMidCategory(name);
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
        button.onclick = () => this.toggleOtherCategories();
        return button;
    },
    
    // 기타 카테고리 토글
    toggleOtherCategories() {
        this.isOtherCategoryOpen = !this.isOtherCategoryOpen;
        const detailSection = document.getElementById('other-categories-detail');
        
        if (this.isOtherCategoryOpen) {
            this.showOtherCategoriesDetail();
            detailSection.style.display = 'block';
        } else {
            detailSection.style.display = 'none';
        }
    },
    
    // 기타 카테고리 상세 표시
    showOtherCategoriesDetail() {
        const container = document.querySelector('.other-category-buttons');
        container.innerHTML = '';
        
        const otherCats = ACIUApp.data.otherCategories[this.selectedLargeCategory] || [];
        otherCats.forEach(cat => {
            const button = document.createElement('button');
            button.className = 'other-category-button';
            button.innerHTML = `${cat.name} (${cat.count})`;
            button.onclick = () => this.selectMidCategory(cat.name);
            container.appendChild(button);
        });
    },
    
    // 중분류 선택 처리
    selectMidCategory(categoryName) {
        this.selectedMidCategory = categoryName;
        this.updateButtonStates();
        this.updateBreadcrumb();
        this.startQuestionMode();
    },
    
    // 버튼 상태 업데이트
    updateButtonStates() {
        // 모든 버튼의 active 클래스 제거
        document.querySelectorAll('.main-category-button, .other-category-button').forEach(btn => {
            btn.classList.remove('active');
        });
        
        // 선택된 버튼 활성화
        const selectedBtn = Array.from(document.querySelectorAll('.main-category-button, .other-category-button'))
            .find(btn => btn.textContent.includes(this.selectedMidCategory));
        if (selectedBtn) {
            selectedBtn.classList.add('active');
        }
    }
};
```

## 4. 다음 단계 작업 순서

1. **서대리**: HTML/CSS 구조 구현 (오늘 완료)
2. **서대리**: JavaScript 로직 구현 (내일 오전)
3. **노팀장**: 코드 검토 및 최적화 (내일 오후)
4. **조대표님**: 최종 테스트 및 승인 (내일 저녁)

조대표님, 이 구조로 서대리에게 작업 지시하겠습니다!