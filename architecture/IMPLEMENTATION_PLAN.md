# Nimson DMS - Feature Implementation Plan

**Generated**: May 8, 2026  
**Status**: Tracking in Progress

---

## Quick Summary

| Category | Count |
|----------|-------|
| ✅ Already Implemented | 7 features |
| ❌ Not Implemented | 25+ features |
| ⚠️ Partially Done | 4 features |

---

## Implementation Phases

### Phase 1: Attendance Enhancements (High Priority)

| # | Feature | Effort | Status |
|---|---------|--------|--------|
| 1.1 | Working Beat Field | 4 hours | ❌ |
| 1.2 | Working Town Field | 4 hours | ❌ |
| 1.3 | GPS Location Tracking | 12 hours | ❌ |

**Backend Changes Needed:**
- Add `workingBeat` field to attendance model
- Add `workingTown` field to attendance model  
- Add `latitude`, `longitude` fields to attendance model

**Frontend Changes Needed:**
- Update attendance form with new fields
- Add GPS capture functionality

---

### Phase 2: New Retailer/Outlet Creation (High Priority)

| # | Feature | Effort | Status |
|---|---------|--------|--------|
| 2.1 | New Counter Creation | 20 hours | ❌ |

**Backend Changes Needed:**
- Create new `/retailers` API routes
- Create Retailer model
- Add retailer creation endpoints

**Frontend Changes Needed:**
- Create `/retailers` page
- Create retailer creation form
- Add image upload for outlet

---

### Phase 3: I Am Here Form (High Priority)

| # | Feature | Effort | Status |
|---|---------|--------|--------|
| 3.1 | Complete I Am Here Module | 24 hours | ❌ |

**Backend Changes Needed:**
- Create `/iamhere` API routes
- Create IAmHere model
- Add submission storage

**Frontend Changes Needed:**
- Create `/i-am-here` page
- Add camera integration
- Add GPS capture
- Add history view

---

### Phase 4: Reports Module (High Priority)

| # | Feature | Effort | Status |
|---|---------|--------|--------|
| 4.1 | My Reports - Working Summary | 16 hours | ❌ |
| 4.2 | My Reports - Enhanced Order History | 8 hours | ⚠️ |
| 4.3 | Team Reports - Daily Report | 20 hours | ❌ |
| 4.4 | Team Reports - Monthly Report | 16 hours | ❌ |
| 4.5 | Primary Sales Report | 16 hours | ❌ |
| 4.6 | Secondary Sales Report | 16 hours | ❌ |
| 4.7 | SKU-wise Sales Report | 16 hours | ❌ |
| 4.8 | User-wise Sales Report | 16 hours | ❌ |
| 4.9 | Order Pending Report | 12 hours | ❌ |
| 4.10 | Beat-wise Sales Report | 16 hours | ❌ |
| 4.11 | Order Status Report | 12 hours | ❌ |
| 4.12 | Opening vs Closing Stock | 16 hours | ❌ |

---

### Phase 5: GPS & Location (Medium Priority)

| # | Feature | Effort | Status |
|---|---------|--------|--------|
| 5.1 | GPS Tracking Module | 40 hours | ❌ |

---

### Phase 6: Digital Collateral (Medium Priority)

| # | Feature | Effort | Status |
|---|---------|--------|--------|
| 6.1 | Enhanced Product Catalogs | 24 hours | ⚠️ |
| 6.2 | Promotional Videos | 16 hours | ❌ |
| 6.3 | PDF Brochures | 12 hours | ❌ |
| 6.4 | Product Launch Info | 16 hours | ❌ |

---

### Phase 7: Gamification (Low Priority)

| # | Feature | Effort | Status |
|---|---------|--------|--------|
| 7.1 | Leaderboards | 32 hours | ❌ |

---

## Files to Modify

### Backend
```
backend/src/modules/attendance/attendance.model.ts - Add fields
backend/src/modules/retailer/ (new module)
backend/src/modules/iamhere/ (new module)
backend/src/modules/gps-tracking/ (new module)
backend/src/modules/reports/ (new module)
backend/src/modules/collateral/ (new module)
backend/src/modules/gamification/ (new module)
```

### Frontend
```
frontend/src/app/attendance/attendance-client.tsx - Add fields
frontend/src/app/retailers/ (new)
frontend/src/app/i-am-here/ (new)
frontend/src/app/my-reports/ (new)
frontend/src/app/team-reports/ (new)
frontend/src/app/reports/ (new)
frontend/src/app/gps-tracking/ (new)
frontend/src/app/collateral/ (new)
frontend/src/app/leaderboards/ (new)
```

---

## Total Estimate

| Phase | Hours | Days (8h/day) |
|-------|-------|---------------|
| Phase 1 | 20h | 2.5 days |
| Phase 2 | 20h | 2.5 days |
| Phase 3 | 24h | 3 days |
| Phase 4 | 164h | 20.5 days |
| Phase 5 | 40h | 5 days |
| Phase 6 | 68h | 8.5 days |
| Phase 7 | 32h | 4 days |
| **Total** | **368h** | **~46 days** |

---

## Next Steps

Which phase would you like to start with?

1. **Phase 1**: Attendance Enhancements (Working Beat, Town, GPS)
2. **Phase 2**: New Retailer Creation
3. **Phase 3**: I Am Here Form
4. **Phase 4**: Reports Module

Tell me which phase to start, and I'll begin implementation!