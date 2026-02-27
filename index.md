# 🚀 뉴비용 자동화 운영 매뉴얼

> 카드 · 현금 매출 자동 통합 시스템  
> Dropbox 공동판매 자동 업데이트 · Slack 리포트 자동 발송

---

## 📌 1. 시스템 개요

Sales Bot은 매일 자동으로 다음 작업을 수행합니다.

### ✅ 자동화 범위

- 🧾 키오스크(키움 관리자) 카드매출 다운로드
- 💵 Google Sheets 현금/계좌 매출 연동
- 📊 카드 + 현금 통합 (Merged 데이터 생성)
- ☁ Dropbox 공동판매 엑셀 자동 업데이트
- 📣 Slack Block Kit 리포트 자동 발송
- 🗓 월초 자동 롤오버 (다음 달 파일 생성)

---

## 🖥 2. 시스템 구조

### 2.1 서버 환경

- AWS Lightsail Ubuntu VM
- 24시간 상시 실행
- SSH 터미널로 접속하여 관리(황태훈만 접속가능)

프로젝트 경로:

```bash
/home/ubuntu/work_automation
```

---

### 2.2 주요 파일 구조

```bash
work_automation/
│
├── run_sales.sh                # 전체 자동 실행 스크립트
├── auto_bot.py                 # 키움 카드매출 다운로드
├── scripts/
│   ├── build_merged_sales.py   # 카드+현금 통합
│   ├── update_joint_sheet.py   # Dropbox 업데이트 + Slack 발송
│   └── rollover_monthly_joint_sheet.py  # 월초 롤오버
│
├── jobs/
│   ├── extract_card.py
│   ├── extract_cash.py
│   ├── merge_sales.py
│   ├── joint_sheet_updater.py
│   ├── joint_sheet_rollover.py
│   └── dropbox_client.py
│
├── downloads/
│   └── archive/YYYYMMDD/
│
├── .env
└── .venv/
```

---

## 🔄 3. 자동 실행 구조

### 3.1 systemd Timer

자동 실행은 `sales-bot.timer`가 담당합니다.

확인:

```bash
systemctl list-timers | grep sales-bot
```

---

## ⏰ 4. 실행 시간 변경

```bash
sudo nano /etc/systemd/system/sales-bot.timer
```

예:

```ini
OnCalendar=*-*-* 00:10:00
```

적용:

```bash
sudo systemctl daemon-reload
sudo systemctl restart sales-bot.timer
```

---

## 📊 5. 매출 데이터 구조

### 카드매출

- 키움 관리자 엑셀 다운로드
- 매장명 1/2 구분 자동 통합

### 현금/계좌

Google Sheets 구조:

| 날짜 | 지점명 | 바코드 | 상품명 | 판매가 | 결제수단 |
|------|--------|--------|--------|--------|----------|

✔ 분할 결제는 행을 2개로 나누어 입력 권장

---

## 🏷 6. 카테고리 집계

### 일반 vs 온라인

- `온`으로 시작 → 온라인
- 그 외 → 일반
- 표시 시 `온` 제거

### 제외 규칙

- 공란
- 미분류
- C
- 위탁

---

## ☁ 7. Dropbox 구조

기본 경로:

```bash
/공동판매_auto/YYYY/M월/
```

`.env` 설정:

```env
DROPBOX_BASE_FOLDER=/공동판매_auto
```

---

## 📣 8. Slack 발송 방식

Incoming Webhook 사용

채널 변경 방법:

1. Slack → Incoming Webhooks
2. 새 Webhook 생성
3. `.env`의 SLACK_WEBHOOK_URL 교체

---

## 🧪 9. 수동 실행

```bash
cd /home/ubuntu/work_automation
source .venv/bin/activate
./run_sales.sh
```

특정 날짜 테스트:

```bash
DATE_YYYYMMDD=20260215 ./run_sales.sh
```

---

## 🔐 10. 보안

`.env`는 반드시 `.gitignore`에 포함되어야 합니다.

확인:

```bash
cat .gitignore | grep .env
```

민감 정보:

- KIWOOM_USERPASS
- SLACK_WEBHOOK_URL
- DROPBOX_REFRESH_TOKEN
- GOOGLE_SERVICE_ACCOUNT_JSON

---

## 📋 11. 운영 체크리스트

### 매일

- Slack 리포트 확인
- Dropbox 파일 업데이트 확인

### 매월 1일

- 전월 매출 반영 확인
- 다음달 파일 생성 확인

---

## 🎯 시스템 상태

- ✔ 일일 자동화 안정화
- ✔ 월초 롤오버
- ✔ 온라인/일반 분리
- ✔ Slack Block Kit 적용
- ✔ 과거 데이터 보호

---

_문의는 황태훈에게..._
