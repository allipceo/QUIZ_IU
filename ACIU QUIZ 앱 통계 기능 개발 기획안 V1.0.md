AICU QUIZ 앱 통계 기능 개발 기획안 V1.0
📊 프로젝트 개요
목표: index_test_v1.0.html 앱에 조대표님의 상세 통계 요구사항을 반영하여 학습 현황을 시각적으로 명확하게 제공하고, 사용자 경험을 향상시킵니다.

핵심 원칙:

코드 최소화: 기존 함수 재활용 및 신규 함수는 엄격히 모듈화하여 2,000줄 이상의 코드 라인 수를 효율적으로 관리합니다.

데이터 지속성: 사용자가 어떤 디바이스에서 접근하더라도 통계 수치가 계속 업데이트되도록 Local Storage를 활용합니다.

사용자 중심: 설정에서 사용자 등록 시점부터 통계 기록을 개시하며, 초기화 기능을 통해 모든 데이터를 관리할 수 있도록 합니다.

🏗️ 시스템 구성 및 데이터 관리
1. 데이터 저장 구조 (Local Storage)
기존 aicu_save_data 및 aicu_user_settings 외에, 통계 데이터를 보다 세분화하여 저장할 수 있도록 구조를 확장합니다.

aicu_statistics_global: 전체 앱에 대한 누적 통계

totalSolved: 총 푼 문제 수

totalCorrect: 총 정답 수

totalIncorrect: 총 오답 수

lastUpdated: 마지막 업데이트 시점 (ISO String)

registrationDate: 사용자 등록 시점 (통계 기록 개시 시점)

aicu_statistics_daily: 일별 통계 (날짜별로 관리)

YYYY-MM-DD: { solved: 오늘 푼 문제 수, correct: 오늘 정답 수, incorrect: 오늘 오답 수 }

aicu_statistics_mode: 학습 모드별 통계 (기본학습, 대분류학습)

basic: { totalSolved, totalCorrect, totalIncorrect }

largeCategory: { totalSolved, totalCorrect, totalIncorrect }

aicu_statistics_category: 대분류 카테고리별 통계 (재산보험, 특종보험 등)

재산보험: { totalSolved, totalCorrect, totalIncorrect, dailySolved, dailyCorrect, dailyIncorrect }

특종보험: { totalSolved, totalCorrect, totalIncorrect, dailySolved, dailyCorrect, dailyIncorrect }

... (나머지 카테고리 동일)

2. 데이터 기록 시점
사용자 등록 시점: saveUserSettings() 함수 내에서 aicu_statistics_global의 registrationDate를 현재 시점으로 기록합니다. 이 시점부터 모든 통계 기록이 개시됩니다.

문제 풀이 시점: checkAnswer() 함수 내에서 정답/오답 여부에 따라 aicu_statistics_global, aicu_statistics_daily, aicu_statistics_mode, aicu_statistics_category 데이터를 실시간으로 업데이트합니다.

🔧 핵심 기능 구현 방안
1. 대문 (Home Screen) 통계
현재 index_test_v1.0.html의 대문 섹션에 통계 정보를 표시합니다.

1.1. 보유 문제수 현황:

ins_master_db.csv에서 "인스교재" 및 "보험중개사 시험"에 해당하는 행 수를 카운트하여 표시합니다.

기존 함수 활용: Papa.parse를 통해 ins_master_db.csv를 로드하는 함수를 재활용하거나, 앱 초기 로딩 시 한 번만 계산하여 전역 변수에 저장합니다.

표시 위치: 현재 "보유 문제수 현황" 섹션 (id="ins-questions", id="exam-questions", id="total-questions")을 업데이트합니다.

1.2. 학습 진도 현황:

완료한 문제: aicu_statistics_global.totalSolved 값을 표시합니다.

전체 진도율: aicu_statistics_global.totalSolved / 총 보유 문제수 (인스교재 + 보험중개사 시험) * 100%를 계산하여 표시합니다.

정답률 (오늘의 진도율 변경): aicu_statistics_global.totalCorrect / aicu_statistics_global.totalSolved * 100%를 계산하여 표시합니다.

표시 위치: 현재 "학습 진도 현황" 섹션 (id="completed-questions", id="overall-progress-rate", id="daily-progress-rate"를 accuracy-rate 등으로 변경)을 업데이트합니다.

1.3. 금일의 학습 현황 박스 (신규 추가):

HTML 구조 추가: 대문 화면에 새로운 박스를 추가하여 "금일의 학습 현황"을 표시합니다.

오늘 풀이한 총 문제수: aicu_statistics_daily[오늘 날짜].solved 값을 표시합니다.

오늘 풀이한 총 문제 중 정답 수: aicu_statistics_daily[오늘 날짜].correct 값을 표시합니다.

오늘 풀이한 전체 문제 대비 정답률: aicu_statistics_daily[오늘 날짜].correct / aicu_statistics_daily[오늘 날짜].solved * 100%를 계산하여 표시합니다.

갱신 시점:

개별 문제 풀이 시 (checkAnswer()) 자동으로 갱신됩니다.

"통계 데이터 불러오기" 버튼 클릭 시 (loadStatisticsData()) 갱신됩니다.

2. 기본학습 (Basic Learning Screen) 통계
기존 basic-learning-screen 내의 통계 영역을 확장합니다.

2.1. 누적 현황:

aicu_statistics_mode.basic.totalSolved (총 문제수)

aicu_statistics_mode.basic.totalCorrect (총 정답수)

aicu_statistics_mode.basic.totalCorrect / aicu_statistics_mode.basic.totalSolved * 100% (총 정답률)

표시 위치: 현재 "학습 통계" 섹션 (id="total-questions", id="correct-answers", id="incorrect-answers")을 재구성하여 표시합니다.

2.2. 금일 현황 (신규 추가):

HTML 구조 추가: 누적 현황 아래에 "금일 현황" 섹션을 추가합니다.

aicu_statistics_daily[오늘 날짜].solved (금일 푼 문제수)

aicu_statistics_daily[오늘 날짜].correct (금일 정답수)

aicu_statistics_daily[오늘 날짜].correct / aicu_statistics_daily[오늘 날짜].solved * 100% (금일 정답률)

갱신 시점: 문제 풀이 시 (checkAnswer()) 및 화면 진입 시 (loadBasicLearningData()) 갱신됩니다.

3. 대분류학습 (Large Category Screen) 통계
기존 large-category-screen 내의 통계 영역을 확장합니다.

3.1. 대분류 학습 전체 누적 통계:

표시 위치: "대분류학습" 제목 우측에 별도 UI 요소를 추가하여 표시합니다.

aicu_statistics_mode.largeCategory.totalSolved (푼 문제수)

aicu_statistics_mode.largeCategory.totalCorrect (맞은 문제수)

aicu_statistics_mode.largeCategory.totalCorrect / aicu_statistics_mode.largeCategory.totalSolved * 100% (정답률)

갱신 시점: 대분류 학습 모드 진입 시 (selectLargeCategory()) 및 문제 풀이 시 (checkAnswer()) 갱신됩니다.

3.2. 특정 대분류 (재산보험, 특종보험 등) 통계:

HTML 구조 추가: 각 카테고리 선택 버튼 아래 또는 별도 섹션에 해당 카테고리의 통계 박스를 추가합니다.

누적 현황:

aicu_statistics_category[카테고리명].totalSolved (총 문제수)

aicu_statistics_category[카테고리명].totalCorrect (총 정답수)

aicu_statistics_category[카테고리명].totalCorrect / aicu_statistics_category[카테고리명].totalSolved * 100% (총 정답률)

금일 현황:

aicu_statistics_category[카테고리명].dailySolved (금일 푼 문제수)

aicu_statistics_category[카테고리명].dailyCorrect (금일 정답수)

aicu_statistics_category[카테고리명].dailyCorrect / aicu_statistics_category[카테고리명].dailySolved * 100% (금일 정답률)

갱신 시점: 해당 카테고리 선택 시 (selectLargeCategory(category)) 및 해당 카테고리 문제 풀이 시 (checkAnswer()) 갱신됩니다.

4. 설정 (User Settings) 모달 통계 관리
현재 학습 통계 표시: updateCurrentStatsDisplay() 함수를 사용하여 aicu_statistics_global 데이터를 기반으로 "푼 문제: X개, 정답률: Y%"를 표시합니다.

전체 초기화: clearAllStatistics() 함수를 확장하여 aicu_statistics_global, aicu_statistics_daily, aicu_statistics_mode, aicu_statistics_category 등 모든 통계 관련 Local Storage 데이터를 삭제하고, 화면에 표시되는 통계 값들을 0으로 초기화합니다.

🛠️ 함수 재활용 및 신규 함수 설계 (모듈화)
기존 ACIU_FINAL_개발경과보고서V1.0.md에 명시된 함수들을 최대한 재활용하고, 필요한 경우 새로운 함수를 추가하되, 기능별로 명확히 분리하여 모듈성을 유지합니다.

1. 기존 함수 확장/수정
updateStatistics(isCorrect, mode, category):

매개변수를 추가하여 현재 학습 모드와 카테고리 정보를 받습니다.

이 함수 내에서 aicu_statistics_global, aicu_statistics_daily, aicu_statistics_mode, aicu_statistics_category 데이터를 모두 업데이트하도록 로직을 확장합니다.

saveProgress() (혹은 새로운 통계 저장 함수)를 호출하여 Local Storage에 변경사항을 즉시 반영합니다.

loadStatisticsData():

대문 화면의 "통계 데이터 불러오기" 버튼 클릭 시 호출됩니다.

Local Storage에서 모든 통계 데이터를 로드하여 전역 window.statistics 객체 및 각 통계 표시 UI에 반영하는 역할을 합니다.

updateQuestionCounts(), updateProgressData(), updateDailyStatisticsDisplay() (신규), updateModeStatisticsDisplay() (신규), updateCategoryStatisticsDisplay() (신규) 등을 호출하여 모든 통계 UI를 갱신합니다.

updateProgressData():

기존 함수를 확장하여 aicu_statistics_global 데이터를 기반으로 "완료한 문제", "전체 진도율", "정답률"을 계산하고 대문에 표시합니다.

updateBasicStatistics() / updateLargeCategoryStatistics():

각 학습 모드 화면에서 호출되며, 해당 모드의 누적 통계(aicu_statistics_mode)를 기반으로 UI를 갱신하도록 로직을 수정합니다.

2. 신규 함수 (필요시)
initializeStatistics():

앱 초기 로딩 시 또는 clearAllStatistics() 호출 시 모든 통계 데이터를 0으로 초기화하고 Local Storage에 저장합니다.

사용자 등록 시 registrationDate를 설정하고, 이 날짜 이전의 통계는 기록하지 않도록 로직을 포함합니다.

updateDailyStatisticsDisplay():

aicu_statistics_daily 데이터를 기반으로 "금일의 학습 현황" 박스를 업데이트하는 전용 함수.

updateModeStatisticsDisplay(mode):

특정 학습 모드(기본학습, 대분류학습)의 통계(aicu_statistics_mode)를 UI에 반영하는 함수.

updateCategoryStatisticsDisplay(category):

특정 대분류 카테고리의 통계(aicu_statistics_category)를 UI에 반영하는 함수.

getTodayDateString():

오늘 날짜를 YYYY-MM-DD 형식의 문자열로 반환하는 유틸리티 함수. 일별 통계 관리에 사용됩니다.

🚀 개발 로드맵 (Phase별)
Phase 1: 데이터 모델 및 초기화 (1일)
Local Storage에 통계 데이터(aicu_statistics_global, aicu_statistics_daily, aicu_statistics_mode, aicu_statistics_category)를 저장할 초기 구조 정의.

initializeStatistics() 함수 구현 및 앱 초기 로딩 시 호출.

saveUserSettings() 함수에 registrationDate 기록 로직 추가.

clearAllStatistics() 함수 확장하여 모든 통계 데이터 초기화 기능 구현.

Phase 2: 대문 통계 구현 (2일)
"보유 문제수 현황" 섹션 업데이트 로직 구현. (CSV 파싱 로직 재활용)

"학습 진도 현황" 섹션 업데이트 로직 구현 (완료한 문제, 전체 진도율, 정답률).

"금일의 학습 현황" 박스 HTML 구조 추가 및 updateDailyStatisticsDisplay() 함수 구현.

checkAnswer() 함수 내에서 대문 통계 데이터 업데이트 로직 연동.

"통계 데이터 불러오기" 버튼 (loadStatisticsData()) 기능 구현 및 연동.

Phase 3: 학습 모드별 통계 구현 (3일)
기본학습:

"누적 현황" 및 "금일 현황" 섹션 HTML 구조 추가/수정.

updateBasicStatistics() 함수 확장하여 해당 UI 업데이트.

checkAnswer() 함수 내에서 기본학습 통계 데이터 업데이트 로직 연동.

대분류학습:

"대분류 학습 전체 누적 통계" HTML 구조 추가 및 updateModeStatisticsDisplay('largeCategory') 함수 구현.

각 특정 대분류(재산보험, 특종보험 등) 통계 섹션 HTML 구조 추가.

updateCategoryStatisticsDisplay(category) 함수 구현.

checkAnswer() 함수 내에서 대분류학습 및 특정 카테고리 통계 데이터 업데이트 로직 연동.

Phase 4: 테스트 및 최적화 (1일)
각 통계 기능별 단위 테스트 수행.

통계 데이터가 Local Storage에 정상적으로 저장되고 로드되는지 확인.

코드 중복 제거 및 함수 재활용 여부 최종 검토.

성능 저하 여부 확인 및 필요한 경우 최적화.

📈 예상 성과 지표
기능 완성도: 통계 요구사항 100% 구현

코드 품질:

신규 코드 라인 최소화 (기존 코드 대비 10% 이내 목표)

함수 재활용률 80% 이상

모듈화 원칙 준수

데이터 지속성: Local Storage를 통한 영구 데이터 저장 및 디바이스 간 동기화 (간접)

사용자 경험: 학습 현황에 대한 명확하고 실시간적인 피드백 제공

⚠️ 고려사항 및 리스크
데이터 정합성: checkAnswer() 함수 내에서 여러 통계 데이터를 동시에 업데이트할 때 데이터 정합성 유지에 주의해야 합니다. 트랜잭션 개념은 아니지만, 순차적으로 업데이트하여 오류를 방지합니다.

Local Storage 용량: 대량의 일별 통계 데이터가 누적될 경우 Local Storage 용량 제한에 유의해야 합니다. (일반적으로 5~10MB) 현재 앱 규모에서는 큰 문제가 없을 것으로 예상되나, 장기적으로는 서버 DB 연동을 고려할 수 있습니다.

날짜 기반 통계: 일별 통계는 사용자의 디바이스 날짜 설정에 의존하므로, 날짜 변경 시 통계가 초기화되는 것처럼 보일 수 있습니다. (이는 요구사항에 부합)

UI 업데이트 성능: 통계 데이터가 많아질 경우 UI 업데이트 시 성능 저하가 발생할 수 있으므로, 효율적인 DOM 조작 방안을 고려합니다.

조대표님, 이 기획안을 바탕으로 통계 기능 개발을 진행하시면 코드 품질과 기능 완성도 모두를 만족시킬 수 있을 것입니다. 추가적인 의견이나 수정사항이 있으시면 언제든지 말씀해 주십시오.