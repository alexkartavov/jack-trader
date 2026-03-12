# HEARTBEAT.md

## Periodic checks (run 2-4x per day)

### pm-trading system health
Check every heartbeat:
```
docker exec pm_worker python3 -c "
from app.database import SessionLocal
from app.models import SystemEvent, Trade, Market
from sqlalchemy import select, desc, func
from datetime import datetime, timezone, timedelta

db = SessionLocal()
now = datetime.now(timezone.utc)
since = now - timedelta(hours=6)

# Recent scan results
events = db.execute(
    select(SystemEvent)
    .where(SystemEvent.event_type == 'scan_complete', SystemEvent.created_at >= since)
    .order_by(desc(SystemEvent.created_at)).limit(3)
).scalars().all()
for e in events:
    print('SCAN:', e.message, '|', e.created_at.strftime('%H:%M'))

# Open trades
open_trades = db.execute(select(func.count()).where(Trade.status=='open')).scalar()
print('Open trades:', open_trades)

# Recent errors
errors = db.execute(
    select(SystemEvent)
    .where(SystemEvent.severity.in_(['error','warning']), SystemEvent.created_at >= since)
    .order_by(desc(SystemEvent.created_at)).limit(5)
).scalars().all()
for e in errors:
    print('ERR:', e.message[:80])

db.close()
"
```

### Alert thresholds (investigate + message Alex if triggered):
- Kalshi markets scanned = 0 → API format change or auth issue
- Polymarket markets scanned = 0 → API down (non-critical)
- No scan_complete event in last 2 hours → scheduler dead
- Open trades > 5 → something approving too aggressively
- Error events mentioning "401" or "403" → Kalshi auth broken
- Any "Connection refused" errors → Docker container issue
