AICU QUIZ 앱 통계 기능 개발 기획서 V2.0
📊 프로젝트 개요
대상 파일: index_test_v1.0.html

현재 코드 라인: 2,000+ 줄 (최소한의 추가 및 기존 함수 재활용)

목표: 조대표님의 상세 통계 요구사항을 반영하여 학습 현황을 시각적으로 명확하게 제공하고, 사용자 경험을 향상시키며, 견고하고 효율적인 통계 시스템을 구축합니다.

핵심 원칙:

코드 효율성: 기존 함수 재활용 및 신규 함수는 엄격히 모듈화하여 코드 라인 수를 효율적으로 관리합니다.

데이터 지속성 및 일관성: 통합 스키마와 Local Storage를 활용하여 어떤 디바이스에서 접근하더라도 통계 수치가 지속적으로 업데이트되고 일관성을 유지하도록 합니다.

사용자 중심: 설정에서 사용자 등록 시점부터 통계 기록을 개시하며, 초기화 및 백업/복구 기능을 통해 데이터 관리 편의성을 제공합니다.

성능 최적화: 캐싱, 배치 업데이트, 지연 업데이트 전략을 통해 앱의 반응성을 유지합니다.

🏗️ 시스템 구성 및 데이터 관리
1. 데이터 저장 구조 (Local Storage)
기존 aicu_save_data, aicu_user_settings 외에, 모든 통계 데이터를 aicu_statistics라는 단일 Local Storage 키 아래 통합 객체로 저장하여 관리 복잡도를 감소시키고 데이터 일관성을 보장합니다.

aicu_statistics 스키마:

const unifiedStatisticsSchema = {
  // 메타 정보
  meta: {
    version: "2.0", // 스키마 버전
    registeredAt: "ISO_STRING", // 사용자 등록 시점 (통계 기록 개시 시점)
    lastUpdated: "ISO_STRING",  // 마지막 통계 업데이트 시점
    userName: "사용자_이름"
  },

  // 모든 통계를 하나의 'stats' 객체로 통합
  stats: {
    // 글로벌 통계 (전체 앱 누적 및 일별)
    global: {
      cumulative: { attempted: 0, correct: 0, accuracy: 0 }, // 누적
      daily: { attempted: 0, correct: 0, accuracy: 0 }       // 오늘
    },

    // 모드별 통계 (기본학습, 대분류학습)
    modes: {
      basic: {
        cumulative: { attempted: 0, correct: 0, accuracy: 0 },
        daily: { attempted: 0, correct: 0, accuracy: 0 }
      },
      largeCategory: {
        cumulative: { attempted: 0, correct: 0, accuracy: 0 },
        daily: { attempted: 0, correct: 0, accuracy: 0 },

        // 카테고리별 통계도 여기에 포함
        categories: {
          "재산보험": {
            cumulative: { attempted: 0, correct: 0, accuracy: 0 },
            daily: { attempted: 0, correct: 0, accuracy: 0 }
          },
          "특종보험": { /* 동일 구조 */ },
          "배상책임보험": { /* 동일 구조 */ },
          "해상보험": { /* 동일 구조 */ }
        }
      }
    }
  },

  // 일별 히스토리 (최근 30일만 보관, 과거 데이터 정리)
  dailyHistory: {
    "YYYY-MM-DD": { attempted: 0, correct: 0, accuracy: 0 },
    // ...최대 30일 데이터 보관
  },

  // 문제 수 캐시 (ins_master_db.csv 파싱 결과, 성능 최적화)
  questionCounts: {
    "인스교재": 0,
    "보험중개사시험": 0,
    "total": 0,
    lastCounted: "ISO_STRING" // 마지막 카운트 시점
  }
};

2. 데이터 기록 시점
사용자 등록 시점: saveUserSettings() 함수 내에서 aicu_statistics.meta.registeredAt을 현재 시점으로 기록하고, 통계 객체를 초기화(또는 기존 데이터 마이그레이션)합니다.

문제 풀이 시점: checkAnswer() 함수 내에서 정답/오답 여부에 따라 aicu_statistics.stats (global, modes, categories) 및 aicu_statistics.dailyHistory 데이터를 실시간으로 업데이트합니다. aicu_statistics.meta.lastUpdated도 갱신합니다.

일별 데이터 자동 정리: cleanupOldDailyData() 함수를 통해 dailyHistory에서 30일 이상 된 데이터를 주기적으로 정리합니다.

🔧 핵심 기능 구현 방안
1. 대문 (Home Screen) 통계
1.1. 보유 문제수 현황:

ins_master_db.csv를 파싱하여 "인스교재", "보험중개사 시험" 및 "합계" 문제 수를 aicu_statistics.questionCounts에 저장하고 UI에 표시합니다. (앱 초기 로드 시 1회 계산 및 캐싱)

1.2. 학습 진도 현황:

aicu_statistics.stats.global.cumulative를 기반으로 "완료한 문제", "전체 진도율", "정답률"을 계산하여 표시합니다. (overall-accuracy-rate ID 사용)

1.3. 금일의 학습 현황 박스 (신규 추가):

대문 화면에 새로운 HTML 박스를 추가하여 aicu_statistics.stats.global.daily 데이터를 기반으로 "오늘 풀이한 총 문제수", "오늘 정답 수", "오늘 정답률"을 표시합니다.

갱신 시점: 문제 풀이 시 자동 갱신 및 "통계 데이터 불러오기" 버튼 클릭 시 갱신됩니다.

2. 기본학습 (Basic Learning Screen) 통계
2.1. 누적 현황: aicu_statistics.stats.modes.basic.cumulative 데이터를 기반으로 "총 문제수", "총 정답수", "총 정답률"을 표시합니다.

2.2. 금일 현황 (신규 추가): aicu_statistics.stats.modes.basic.daily 데이터를 기반으로 "금일 푼 문제수", "금일 정답수", "금일 정답률"을 표시합니다.

갱신 시점: 문제 풀이 시 자동 갱신 및 기본학습 화면 진입 시 갱신됩니다.

3. 대분류학습 (Large Category Screen) 통계
3.1. 대분류 학습 전체 누적 통계:

"대분류학습" 제목 우측에 aicu_statistics.stats.modes.largeCategory.cumulative 데이터를 기반으로 "푼 문제수", "맞은 문제수", "정답률"을 표시합니다.

3.2. 특정 대분류 (재산보험, 특종보험 등) 통계:

각 카테고리 선택 시, 해당 카테고리(aicu_statistics.stats.modes.largeCategory.categories[카테고리명])의 "누적 현황" 및 "금일 현황"을 표시합니다.

갱신 시점: 대분류 학습 모드 진입 시 및 특정 카테고리 선택/문제 풀이 시 갱신됩니다.

4. 설정 (User Settings) 모달 통계 관리
현재 학습 통계 표시: updateCurrentStatsDisplay() 함수를 사용하여 aicu_statistics.stats.global.cumulative 데이터를 기반으로 "푼 문제: X개, 정답률: Y%"를 표시합니다.

전체 초기화: clearAllStatistics() 함수를 확장하여 aicu_statistics 객체 전체를 초기화하고 Local Storage에서 삭제합니다.

데이터 백업/복구 기능 (추가): 사용자가 통계 데이터를 JSON 파일로 내보내거나 가져올 수 있는 기능을 추가하여 데이터 손실 위험을 줄입니다.

🛠️ 핵심 함수 설계 및 개선
기존 함수를 확장/수정하고, 노팀장님과 서대리님의 제안을 반영하여 통합적이고 효율적인 함수 구조를 구축합니다.

1. 데이터 관리 함수
initializeStatistics(): 앱 초기 로딩 시 또는 clearAllStatistics() 호출 시 aicu_statistics 객체를 초기화하고 Local Storage에 저장합니다. (사용자 등록 시점 로직 연동)

loadStatisticsData(): Local Storage에서 aicu_statistics를 로드하고, StatisticsCache를 통해 메모리에 캐싱합니다.

saveStatisticsData(): StatisticsCache의 flush() 메서드를 호출하여 캐싱된 데이터를 Local Storage에 저장합니다.

clearAllStatistics(): 모든 통계 데이터 초기화 및 UI 리셋.

getTodayKey(): 오늘 날짜를 YYYY-MM-DD 형식으로 반환하는 유틸리티.

cleanupOldDailyData(stats): dailyHistory에서 30일 이상 된 데이터를 자동 정리.

migrateOldData(): 기존 aicu_save_data를 새로운 aicu_statistics 스키마로 변환하는 마이그레이션 로직 (앱 최초 실행 시 1회).

2. 통계 계산 및 업데이트 함수 (통합 및 최적화)
updateAllStatistics(isCorrect, mode, category): (노팀장 제안 반영)

단일 함수로 모든 통계 (global, modes, categories, dailyHistory)를 업데이트합니다.

StatisticsCache에서 데이터를 가져와 메모리에서 계산을 수행하고, isDirty 플래그를 설정합니다.

호출 시점: checkAnswer() 함수 내에서 호출.

calculateAccuracy(correct, total): (서대리 제안 반영)

캐싱을 적용하여 중복 계산을 방지하고 성능을 최적화합니다.

calculateProgress(completed, total): 진도율 계산.

countQuestionsBySource(csvData): ins_master_db.csv에서 문제 수를 카운트.

3. UI 업데이트 함수 (배치 처리 및 사용자 경험 개선)
updateAllUI(stats): (노팀장 제안 반영)

모든 통계 관련 UI 요소를 한 번에 업데이트하여 DOM 조작 횟수를 최소화하고 성능을 향상시킵니다.

updateBatchDOM(updates): 실제 DOM 업데이트를 수행하는 헬퍼 함수.

updateHomeStatistics(): 대문 통계 UI 업데이트.

updateBasicLearningStats(): 기본학습 통계 UI 업데이트.

updateLargeCategoryStats(category): 대분류 학습 전체 및 카테고리별 통계 UI 업데이트.

showStatisticsLoading() / hideStatisticsLoading(): (노팀장 제안 반영)

통계 로딩 중 스켈레톤 UI를 표시하여 사용자에게 시각적 피드백을 제공합니다.

animateNumber(element, from, to, duration): (서대리 제안 반영)

숫자 변화에 애니메이션 효과를 적용하여 사용자 경험을 향상시킵니다.

4. 성능 최적화 및 안정성
StatisticsCache 클래스 (노팀장 제안 반영):

Local Storage 접근을 최소화하고 메모리 캐시를 활용하여 성능을 대폭 향상시킵니다. get(), set(), flush() 메서드를 포함합니다.

지연 업데이트 (scheduleUIUpdate()):

답안 제출 시 통계 데이터는 즉시 업데이트하되, UI 업데이트는 짧은 지연(예: 100ms) 후 배치로 처리하여 불필요한 리렌더링을 줄입니다.

validateStatistics(stats): (서대리 제안 반영)

데이터 무결성 검증 로직을 추가하여 통계 데이터의 신뢰성을 확보합니다.

updateStatisticsAtomically(updates): (서대리 제안 반영)

통계 업데이트 시 원자성을 보장하는 트랜잭션적 접근을 고려합니다. (필요시 구현)

🚀 개발 로드맵 (Phase별)
Phase 1: 핵심 인프라 (1일)

aicu_statistics 통합 스키마 정의 및 StatisticsCache 클래스 구현.

initializeStatistics(), loadStatisticsData(), saveStatisticsData() 등 기본 CRUD 함수 구현.

migrateOldData() 함수 구현 및 앱 초기 로딩 시 실행.

countQuestionsBySource() 함수 구현 및 보유 문제수 캐싱.

Phase 2: 대문 통계 (1일)

"보유 문제수 현황", "학습 진도 현황" UI 업데이트 및 데이터 연동.

"금일 학습현황" 박스 HTML 추가 및 updateDailyStatisticsDisplay() 함수 구현.

updateAllStatistics() 함수 내 대문 통계 업데이트 로직 연동.

"통계 데이터 불러오기" 버튼 (loadStatisticsData()) 기능 연동.

Phase 3: 학습 모드 통계 (1일)

기본학습 화면에 누적/금일 통계 UI 추가 및 updateBasicLearningStats() 연동.

대분류학습 화면에 전체 누적 통계 및 카테고리별 상세 통계 UI 추가.

updateLargeCategoryStats() 및 updateCategoryStatisticsDisplay() 연동.

updateAllStatistics() 함수 내 모드별/카테고리별 통계 업데이트 로직 연동.

Phase 4: 최적화 및 폴리싱 (0.5일)

updateAllUI() 및 scheduleUIUpdate()를 통한 배치 UI 업데이트 적용.

animateNumber()를 활용한 숫자 변화 애니메이션 적용.

showStatisticsLoading() / hideStatisticsLoading() 로딩 상태 표시 구현.

cleanupOldDailyData() 구현 및 주기적 실행 로직 추가.

데이터 백업/복구 기능 (exportStatistics, importStatistics) 구현.

전반적인 버그 수정 및 코드 정리.

📈 예상 성과 지표
기능 완성도: 조대표님의 모든 통계 요구사항 100% 구현.

코드 품질:

신규 코드 라인 최소화 (기존 코드 대비 약 500줄 증가 예상)

함수 재활용률 80% 이상 및 모듈화 원칙 준수.

데이터 지속성: Local Storage를 통한 영구 데이터 저장 및 높은 데이터 일관성.

성능:

통계 업데이트 시간 < 100ms (UI 배치 업데이트 기준).

메모리 사용량 효율적 관리 (캐싱 및 정리).

사용자 경험: 직관적인 통계 표시, 로딩 지연 최소화, 애니메이션을 통한 시각적 만족도 향상, 데이터 손실 방지.

⚠️ 고려사항 및 리스크
데이터 마이그레이션: 기존 사용자의 aicu_save_data를 새로운 aicu_statistics 스키마로 안전하게 변환하는 로직의 견고성 확보. (Phase 1에서 집중)

Local Storage 용량: 일별 통계 데이터의 30일 보관 정책으로 용량 문제는 크게 없을 것으로 예상되나, 지속적인 모니터링 필요. (장기적으로 클라우드 DB 연동 고려)

UI/UX 일관성: 다양한 통계 섹션 추가에 따른 전반적인 UI/UX 일관성 및 미려함 유지.

오프라인 지원: Local Storage 기반으로 오프라인에서도 통계 기록 및 조회가 가능하나, 크로스 디바이스 동기화는 온라인 환경에서만 가능함을 명확히 안내.

🎯 최종 권장사항
통합 스키마 (aicu_statistics) 채택: 데이터 관리 복잡도와 일관성 측면에서 가장 효율적입니다.

StatisticsCache 클래스 필수 적용: Local Storage 접근을 최소화하여 성능을 최적화하는 핵심 요소입니다.

updateAllStatistics() 및 updateAllUI()를 통한 배치 처리: 통계 업데이트 및 UI 렌더링 성능을 극대화하고 사용자 경험을 향상시킵니다.

데이터 백업/복구 기능 제공: 사용자에게 심리적 안정감을 제공하고 데이터 손실 위험을 줄입니다.

점진적 구현 및 테스트: 각 Phase별로 기능을 구현하고 철저히 테스트하여 안정성을 확보합니다.

이 기획서 V2.0은 조대표님의 요구사항을 완벽히 충족시키면서도, 기술적 완성도와 사용자 경험을 동시에 고려한 최적의 방안이라고 확신합니다.