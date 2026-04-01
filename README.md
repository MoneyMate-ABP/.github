# Expense Tracker Description

### Tech Stack
| Layer | Tech |
|-------|------|
| Backend | Express.js + JWT |
| Web | ReactJS (PWA) |
| Mobile | Flutter |
| Database | PostgreSQL / MySQL |

---

### Fitur Final

#### 1. Auth
- Register, Login, Logout
- Token-based (JWT)

#### 2. Kategori
- Default: Makanan, Transportasi, Hiburan, Lainnya
- Pilih kategori saat input transaksi

#### 3. Transaksi
- Tambah / Edit / Hapus
- Tipe: income / expense
- Filter by tanggal & tipe
- Simpan latitude & longitude (opsional)

#### 4. Budget Period
- **Bisa multiple budget aktif sekaligus** (per kategori atau global)
- Set: total budget, start date, end date
- Auto-hitung working days (exclude Sabtu & Minggu)
- Auto-hitung daily budget base

#### 5. Daily Budget Tracker
- Budget efektif = base + carry over kumulatif
- Surplus → nambah budget hari berikutnya
- Deficit → kurangi budget hari berikutnya
- Weekend: budget base = 0, tapi carry over tetap jalan
- **Hitung realtime** (tidak disimpan ke DB)

#### 6. Dashboard
- Total saldo, pemasukan, pengeluaran
- Budget efektif hari ini per budget period
- Sisa / surplus / deficit hari ini

#### 7. Bonus
- Notifikasi harian jam 20:00
- Geolocation saat input transaksi

---

### Database Schema Final

```sql
-- users
id, name, email, password, created_at

-- categories
id, name, type (income/expense/both)

-- transactions
id, user_id, category_id, budget_period_id (nullable),
type, amount, note, date, latitude, longitude, created_at

-- budget_periods
id, user_id, category_id (nullable, null = global),
name, total_budget, start_date, end_date,
daily_budget_base, working_days_count, created_at
```

### Logika Realtime Carry Over (di backend)

```javascript
// Pseudocode endpoint GET /api/budget-periods/:id/daily-status?date=

function getDailyStatus(budgetPeriod, targetDate) {
  const days = getDaysFromStartTo(targetDate) // semua hari sejak start
  
  let carryOver = 0

  for (const day of days) {
    const isWeekend = isWeekend(day)
    const base = isWeekend ? 0 : budgetPeriod.daily_budget_base
    const effectiveBudget = base + carryOver
    const spent = getTotalSpent(day, budgetPeriod) // query transaksi
    carryOver = effectiveBudget - spent // bisa + atau -
  }

  return {
    date: targetDate,
    base: isWeekend(targetDate) ? 0 : budgetPeriod.daily_budget_base,
    carry_over: carryOver sebelum hari ini,
    effective_budget: base + carry_over,
    total_spent: spent hari ini,
    remaining: effective_budget - total_spent
  }
}
```

---

### API Endpoints Lengkap

```
-- Auth --
POST   /api/auth/register
POST   /api/auth/login
POST   /api/auth/logout

-- Transactions --
GET    /api/transactions          ?date=&type=&category=
POST   /api/transactions
PUT    /api/transactions/:id
DELETE /api/transactions/:id

-- Categories --
GET    /api/categories

-- Budget Periods --
GET    /api/budget-periods                        (list semua)
POST   /api/budget-periods                        (buat baru)
PUT    /api/budget-periods/:id
DELETE /api/budget-periods/:id
GET    /api/budget-periods/:id/daily-status       ?date=YYYY-MM-DD

-- Dashboard --
GET    /api/dashboard                             (summary global)
```
