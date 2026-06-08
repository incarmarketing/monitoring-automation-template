# 운영 안정화 메모

이 템플릿은 GitHub Actions, Supabase, cron-job.org를 함께 쓰는 구조를 기준으로 합니다. 실제 운영에서는 아래 문제가 자주 생길 수 있어서 기본 방어 로직을 포함했습니다.

## 포함된 안정화 기능

1. 보고서 중복 실행 방지

- `schedule_guard.py`가 Supabase `job_runs`를 확인합니다.
- 이미 성공한 슬롯은 다시 보내지 않습니다.
- 최근 실패한 슬롯은 기본 30분 동안 재시도하지 않습니다.
- 이미 실행 중인 슬롯은 기본 20분 동안 중복 실행하지 않습니다.

2. 부정/주의 감시 후 대시보드 갱신

- `negative_watch.py`는 감시 결과를 Supabase에 기록합니다.
- 신규 부정/주의 이슈가 있으면 대시보드 갱신 workflow를 깨울 수 있습니다.
- 평상시에는 `NEGATIVE_WATCH_DASHBOARD_REFRESH_MINUTES` 값에 따라 주기적으로 정적 대시보드 스냅샷을 갱신합니다.

3. 금융당국 보도자료 증분 수집

- 매번 오래된 전체 archive를 훑지 않도록 기본값을 `45일 / 10페이지`로 제한했습니다.
- 과거 전체 백필이 필요하면 `Regulator Releases Backfill` workflow를 수동 실행하면서 `days`, `pages` 값을 늘리면 됩니다.

4. 외부 cron 중복 job 정리

- `setup_cronjob_org.py`는 현재 관리 대상 job을 생성/수정합니다.
- 같은 제목으로 중복된 job이나 예전 prefix의 job은 기본적으로 비활성화합니다.
- 비활성화를 끄고 싶으면 `CRONJOB_DISABLE_STALE=false`를 설정합니다.

## 권장 운영값

```text
REPORT_FAILURE_COOLDOWN_MINUTES=30
REPORT_IN_PROGRESS_COOLDOWN_MINUTES=20
NEGATIVE_WATCH_DASHBOARD_REFRESH=throttled
NEGATIVE_WATCH_DASHBOARD_REFRESH_MINUTES=15
CRONJOB_DISABLE_STALE=true
```

## 주의

공유 템플릿에서는 자동 schedule trigger가 기본적으로 꺼져 있습니다. Secrets, Supabase, GitHub Pages, Kakao 설정을 끝낸 뒤에만 workflow schedule을 켜세요.
