# 씬잉글리쉬 영어일기 AI 피드백 앱 — 프로젝트 스펙

## 1. 프로젝트 개요

- **목적**: 학생이 매일 짧은 영어 질문(주제)을 받아 3줄 내외로 답을 쓰면, AI가 문법/표현만 간단히 교정해주는 아티팩트형 웹앱
- **대상**: 씬잉글리쉬 스튜디오 학생 (성인 OPIc 대비반 중심, 추후 키즈반 확장 가능)
- **목표 레벨**: OPIc IH
- **제작 방식**: Claude 아티팩트 (window.storage 사용, 백엔드 별도 구축 없음)
- **브랜드 스타일**: 브랜드컬러 #92308D, Pretendard 폰트, 따뜻하지만 담백한 톤

---

## 2. 사용자 구조

- 회원가입 절차 없이 **이름 + 4자리 PIN**으로 학생 구분
- 최초 접속 시 이름+PIN 조합이 없으면 자동으로 신규 학생 계정 생성
- 학생별 데이터는 개인 저장 영역(shared: false)에 저장 — 학생마다 자기 기록만 보임
- 원장님 전용 뷰(전체 학생 현황 확인용)는 2단계 이후 확장 항목으로 보류

---

## 3. 오늘의 질문 — OPIc 서베이 기반 카테고리 (14개)

OPIc 배경 설문(Background Survey)에서 실제로 선택 가능한 항목을 기반으로 구성. 각 카테고리마다 **일상 묘사 / 경험담 / 비교 / 습관 변화**형 질문을 섞어서, IH 단골 질문 패턴에 자연스럽게 노출되도록 설계.

### ① 거주 형태 (Living Situation)
- What does your home look like?
- Describe your favorite room in your house.
- How is your neighborhood different from other areas in your city?
- Have you moved recently? What changed?

### ② 함께 사는 사람 / 가족 (Family & Housemates)
- Who do you live with?
- Describe a family member you are close to.
- What do you usually do together on weekends?
- How has your relationship with your family changed over time?

### ③ 직업 / 하는 일 (Job & Occupation)
- What do you do for a living?
- Describe a typical day at your job.
- What was your first job like?
- How has your job changed since you started?

### ④ 학생 신분 (School Life — 학생인 경우)
- What subject are you studying these days?
- Describe your classroom or study environment.
- Tell me about a memorable class you took.
- How has your way of studying changed recently?

### ⑤ 여가 활동 (Leisure Activities)
- What do you usually do in your free time?
- Describe your favorite way to relax.
- Tell me about a recent weekend you enjoyed.
- How do you spend your free time differently now compared to a few years ago?

### ⑥ 취미와 관심사 (Hobbies & Interests)
- What is your favorite hobby?
- Describe how you got interested in this hobby.
- Tell me about a time your hobby didn't go as planned.
- How has this hobby changed over the years?

### ⑦ 운동 (Sports & Exercise)
- Do you exercise regularly? What kind?
- Describe the place where you usually exercise.
- Tell me about a memorable experience related to sports.
- Compare how you exercised in the past vs. now.

### ⑧ 국내 여행 (Domestic Travel)
- Describe a place in Korea you like to visit.
- Tell me about your most recent domestic trip.
- What problems have you had while traveling domestically?
- How is domestic travel different now compared to before?

### ⑨ 해외 여행 (International Travel)
- Have you traveled abroad? Where did you go?
- Describe your most memorable international trip.
- Tell me about a problem you experienced while traveling abroad.
- How do you prepare differently for trips now?

### ⑩ 카페 / 외식 (Cafes & Dining Out)
- Do you often go to cafes or restaurants?
- Describe your favorite cafe or restaurant.
- Tell me about a memorable meal you had recently.
- How have your dining habits changed?

### ⑪ 영화 감상 (Watching Movies)
- What kind of movies do you like?
- Describe the last movie you watched.
- Tell me about a movie that left a strong impression on you.
- How has the way you watch movies changed (theater vs. streaming)?

### ⑫ 공연 / 콘서트 관람 (Concerts & Performances)
- Have you been to a concert or performance recently?
- Describe the atmosphere of a performance you attended.
- Tell me about a memorable experience at a concert.
- How is watching a performance different from watching it online?

### ⑬ 독서 (Reading)
- What kind of books do you usually read?
- Describe a book that influenced you.
- Tell me about your reading habits.
- How have your reading habits changed over time?

### ⑭ 음악 감상 (Listening to Music)
- What kind of music do you listen to?
- Describe when and where you usually listen to music.
- Tell me about a memorable experience related to music.
- How has your taste in music changed over the years?

> 확장용 예비 카테고리(필요 시 추가): 국내/해외 출장, 자원봉사, 요리, 반려동물, SNS 사용

---

## 4. 질문 출제 로직

- 카테고리 순서대로 매일 하나씩 순환 (14일 주기)
- 같은 카테고리 안에서는 묘사 → 경험담 → 비교 → 습관변화 순으로 난이도/유형을 다양화
- AI는 완전히 새 질문을 생성하지 않고, **정해둔 문항 뼈대 안에서 표현만 살짝 변주** (품질 편차 방지)

---

## 5. AI 피드백 규칙

- 범위: **문법/표현 교정만 간단히**
- 포함 항목:
  1. 원문
  2. 교정문
  3. 무엇이 왜 틀렸는지 한 줄 설명 (한국어)
- 제외 항목: 레벨 평가, 격려 코멘트, 모범 문장 제시, 감정적 피드백 — 담백하고 짧게 유지

---

## 6. 데이터 모델 (window.storage 키 설계)

```
students:{name}_{pin}       → { name, pin, createdAt, totalDaysWritten, lastWrittenDate, cycleRound }
entries:{name}_{pin}:{date} → { date, category, questionType, question, studentAnswer, aiCorrection }
```
- `cycleRound`: 14일 주기를 몇 바퀴째 돌고 있는지 (1회차/2회차/3회차...) — 회차에 따라 질문 유형 난이도(묘사→돌발질문)를 결정
- 학생별 기록과 누적 작성일수를 한 번에 묶어 저장 (호출 횟수 최소화)

---

## 7. 화면 구성

1. 로그인 (이름 + PIN 입력)
2. 오늘의 질문 화면 (카테고리 표시 + 질문 + 3줄 입력창)
3. 제출 → AI 교정 결과 화면
4. 지난 기록 보기 (달력형, 총 작성일수 누적 표시 — 스트릭 개념 없음)

---

## 8. 배포

- Claude 아티팩트 링크로 학생들에게 공유 (즐겨찾기 저장 안내)
- 추후 독립 앱 필요 시: Anthropic API 키 + 별도 백엔드(Cloudflare Worker 등)로 이관 가능

---

## 9. 결정사항 (확정)

- **14일 주기 이후**: 동일 카테고리를 반복하되, 2회차부터는 유형을 심화 — 기존 묘사/경험담/비교/습관변화에 더해 **돌발 질문형**(예: "만약 ~라면?", "다른 사람과 비교했을 때" 등 즉흥 대응 유형)을 섞어서 난이도를 올림. 3회차 이후도 동일한 방식으로 계속 심화.
- **하루 결석 시**: 스트릭(연속일수) 개념 없이, **총 작성일수만 누적 카운트**로 표시. 하루를 놓쳐도 페널티 없이 이어서 작성 가능.
- **원장님용 전체 학생 현황 대시보드**: 이번 버전에서는 제작하지 않음. 학생 개인용 화면(로그인 → 오늘의 질문 → 작성 → 피드백 → 누적 작성일수)까지만 우선 구현.

> 위 결정에 따라 데이터 모델에서 `currentStreak` 항목은 제거하고 `totalDaysWritten`(총 작성일수)으로 대체.
