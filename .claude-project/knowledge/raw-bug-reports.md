# Raw Bug Reports

> Total: 535 bugs from 15 projects


## [3078] SMS service not set up — phone OTP does not work

- **Status**: Backlog

- **URL**: https://www.notion.so/319b6d88d2cf81288c58e580cef5509b

- **Dev Comment**: Skipped - requires SMS provider business decision (e.g., Twilio, AWS SNS, Korean provider). No code changes needed until provider is selected.



## [3070] Coach signup form does not create an account

- **Status**: Close

- **URL**: https://www.notion.so/319b6d88d2cf81098411f9312390726a

- **Dev Comment**: Already fixed in commit 7d790fe (2026-03-04). The handleSubmit was just calling navigate("/login") without any API call. Fixed by adding authService.register() with name, email, password, role=COACH. Also added loading state, error handling, and success message UI. Fix is on both ashadul.dev and dev branches.



## [3073] Admin patient table — Export button does nothing

- **Status**: Close

- **URL**: https://www.notion.so/319b6d88d2cf8109ad64c3f9840cae48



## [3079] Remove unused counter code (counterSlice and Counter component)

- **Status**: Close

- **URL**: https://www.notion.so/319b6d88d2cf8120b6e0f91326d4a25b

- **Dev Comment**: Removed unused counter boilerplate code. Deleted counterSlice.ts and Counter.tsx, removed counterReducer from rootReducer.ts. Frontend build verified.



## [3072] Admin coach table — Export button does nothing

- **Status**: Close

- **URL**: https://www.notion.so/319b6d88d2cf8127bddffa8ce5d0d78e



## [3071] Coach signup — license number and hospital code fields are never saved

- **Status**: Close

- **URL**: https://www.notion.so/319b6d88d2cf81b58a12e8b6ce3d8a30

- **Dev Comment**: Fixed full-stack pipeline: Added licenseNumber and hospitalCode columns to User entity, RegisterDto, auth.service register method, frontend RegisterRequest type, and coach signup form handleSubmit. Generated migration. Files: user.entity.ts, register.dto.ts, auth.service.ts, auth.d.ts, coach.tsx



## [3080] Coach and patient profile pages — user cannot edit their information

- **Status**: Close

- **URL**: https://www.notion.so/319b6d88d2cf81cd9c40ea07b59f001d



## [3075] Patient notifications — clicking a notification does not navigate anywhere

- **Status**: Close

- **URL**: https://www.notion.so/319b6d88d2cf81e0a7a5d0ae831b05b9



## [3077] Coach patient list — Sort button does nothing

- **Status**: Close

- **URL**: https://www.notion.so/319b6d88d2cf81f39e66ec00182383b2



## [3056] Dynamic input fields based on Role selection in 'Create New User' modal

- **Status**: Close

- **URL**: https://www.notion.so/318b6d88d2cf8088913bff69908daf11



## [3054] backbutton redirection issue

- **Status**: Close

- **URL**: https://www.notion.so/318b6d88d2cf80b6b857f20e86f3f9bd



## [3053] The individual intensity doesn’t show the exercise record in admin dashboard.

- **Status**: Close

- **URL**: https://www.notion.so/318b6d88d2cf807a8430c7dce096bb78

- **Dev Comment**: Fixed: handleSimpleRating in exercise-detail.tsx was ignoring the rating param (prefixed _ as unused). Added RATING_TO_INTENSITY map (1=LOW/Easy, 2=MEDIUM/Normal, 3=HIGH/Difficult) and passed individualIntensity to logExercise.mutateAsync(). Admin panel ExerciseCell already displays intensity badges correctly - no backend changes needed. File: frontend/app/pages/patient/exercise-detail.tsx



## [3049] Layout breaks (Text wraps) when the current date is focused

- **Status**: Close

- **URL**: https://www.notion.so/318b6d88d2cf80589c5af45b25a47b1d



## [3032] Inconsistent Session Bar Width in Calendar

- **Status**: Close

- **URL**: https://www.notion.so/317b6d88d2cf80e18aeccd8371fad1a1

- **Dev Comment**: Fixed by adding display: block to FullCalendar event objects. Timed events in month view were rendering as dot-events (content-width) instead of block-events (full cell width). File: frontend-dashboard/app/components/admin/calendar/CalendarContainer.tsx



## [3031] 'Deselect All' filter not working on Sessions Calendar

- **Status**: Close

- **URL**: https://www.notion.so/317b6d88d2cf8080b7cde44cfface40b

- **Dev Comment**: Fixed two bugs:
1. CalendarContainer.tsx: filteredMeetings returned ALL meetings when selectedCoachIds was empty. Changed to return [] instead.
2. home.tsx: API query fetched all meetings when no coaches selected. Added enabled: selectedCoachIds.length > 0 to skip the request.

Files modified:
- frontend-dashboard/app/components/admin/calendar/CalendarContainer.tsx
- frontend-dashboard/app/pages/admin/home.tsx



## [3030] Fix table header at the top while scrolling

- **Status**: Close

- **URL**: https://www.notion.so/314b6d88d2cf803d8312fc71162faa4f



## [3025] Wrong Redirection for "View Session Records”

- **Status**: Close

- **URL**: https://www.notion.so/314b6d88d2cf80bba89beb9965be016f



## [3024] Add a checkmarker

- **Status**: Close

- **URL**: https://www.notion.so/314b6d88d2cf806fa9d1dcfcce00513b

- **Dev Comment**: Fixed 2 issues:
1. UI: Checkmark icon now displays inside checkbox when checked (fixed Tailwind CSS targeting)
2. Auto-login: Cookie now persists for 30 days when rememberMe is enabled (added maxAge to httpOnly cookie)

Files changed:
- frontend/app/pages/auth/login.tsx
- backend/src/core/interceptors/set-token.interceptor.ts



## [3023] Validation issue on the survey

- **Status**: Close

- **URL**: https://www.notion.so/314b6d88d2cf8093a355d95fedec1522

- **Dev Comment**: Added onKeyDown handlers to both walking minutes and step count inputs to block non-numeric characters (e, E, +, -, .). File: frontend/app/pages/patient/survey.tsx



## [3022] Restrict access to future exercises and prevent editing past records

- **Status**: Close

- **URL**: https://www.notion.so/313b6d88d2cf8028b76cc40dbdd9d173

- **Dev Comment**: Implemented date-based access control:

Backend:
- Added DateUtil.isToday() and assertTodayOnly() guards
- POST /exercise-logs rejects non-today dates (400)
- POST /exercise-days/complete rejects non-today dates (400)
- POST /surveys/daily rejects non-today dates (400)

Frontend (patient home page):
- Future dates: Exercise card shows lock icon, non-clickable. Survey shows Scheduled.
- Past dates: Exercise card is read-only with neutral styling.
- Today: No changes, fully functional.
- Zoom sessions: Remain accessible on all dates.

Files modified:
- backend/src/core/utils/date.util.ts
- backend/src/modules/exercises/services/exercise-log.service.ts
- backend/src/modules/exercises/services/exercise-day.service.ts
- backend/src/modules/surveys/services/daily-survey.service.ts
- frontend/app/pages/patient/home.tsx
- frontend/app/i18n/locales/en.json
- frontend/app/i18n/locales/ko.json



## [3021] Exercise Count Mismatch

- **Status**: Close

- **URL**: https://www.notion.so/313b6d88d2cf80cd84e8ec6da727e5d7

- **Dev Comment**: Fixed getDaySummary() in exercise-day.service.ts to always validate totalCount against current active prescriptions instead of returning stale ExerciseDay records. Now if prescriptions are deactivated/removed, the count correctly reflects 0 and returns null.



## [3017] Date Dividers are Missing

- **Status**: Close

- **URL**: https://www.notion.so/313b6d88d2cf80dbb7d1eb56cee34321

- **Dev Comment**: Added date dividers to chat message list in both patient and coach views.

Files changed:
- NEW: frontend/app/components/chat/chat-date-divider.tsx (ChatDateDivider component + helpers)
- MOD: frontend/app/pages/patient/chat-detail.tsx (date divider rendering)
- MOD: frontend/app/pages/coach/chat-detail.tsx (date divider rendering)

Shows "Today", "Yesterday", or full date (e.g. 2026년 2월 25일) between messages from different days. Supports KO/EN i18n. Build verified.



## [3001] Indicator Mismatch with real data

- **Status**: Close

- **URL**: https://www.notion.so/312b6d88d2cf80949c84f933eb996896

- **Dev Comment**: Fixed: 3 bugs causing calendar dot indicators to not sync with real data.

1. Survey cache invalidation - useSubmitSurvey now invalidates all survey queries (surveyKeys.all) so calendar dots refresh after submission
2. Timezone mismatch - useSurveyStatusByDate now uses local timezone instead of UTC for today detection
3. Date parsing - Month view dot indicators now use string-based date extraction instead of fragile new Date() parsing

Files modified:
- frontend/app/services/httpServices/queries/useSurveys.ts
- frontend/app/pages/patient/home.tsx



## [2997] Fix data sync issues between Calendar and Zoom session info

- **Status**: Close

- **URL**: https://www.notion.so/311b6d88d2cf8073b8b1d3dd0f20d77f

- **Dev Comment**: ## Fix Implemented

**Files Modified:**
- backend/src/modules/meetings/repositories/zoom-meeting.repository.ts
- backend/src/modules/exercises/services/exercise-day.service.ts
- frontend/app/pages/patient/home.tsx

**Changes:**
1. [Backend] zoom-meeting.repository.ts: Fixed findByPatientAndDateRange and findByCoachAndDateRange to normalize dates to start-of-day (00:00:00.000) and end-of-day (23:59:59.999) before querying with Between(). Previously, passing the same date as both startDate and endDate would only match meetings at exactly midnight UTC, missing all real meetings on that day.

2. [Backend] exercise-day.service.ts: Modified getDaySummary to return a synthetic summary object with prescription-based totalCount and log-based completedCount when no ExerciseDay record exists for the selected date. Previously returned null for any date where the patient had not pressed the complete button, causing the exercise card to show 0/0 completed with 0% progress even when active prescriptions existed.

3. [Frontend] home.tsx: Stabilized the today reference using useMemo(() => new Date(), []) to prevent unnecessary re-renders caused by a new Date object being created on every render cycle.

**Pages Affected:** /patient (home page)

**Commit:** fix: correct date-based schedule display on patient home page

---
Updated by Claude Code



## [2995] Add the Eng language in this app

- **Status**: Close

- **URL**: https://www.notion.so/311b6d88d2cf80c487c6c571fba3a3d2



## [2994] The word displayed is incorrect

- **Status**: Close

- **URL**: https://www.notion.so/311b6d88d2cf80539bdbc7587d32fa53

- **Dev Comment**: Fix implemented: Changed exercise button text logic in patient home page. When today is selected and no exercises have been started (completedCount === 0), the button now shows '시작하기' instead of '이어서 하기'. '이어서 하기' is only shown when completedCount > 0 (user has started at least one exercise). File: frontend/app/pages/patient/home.tsx line 519.



## [2993] Exercise count is not updated on the home screen

- **Status**: Close

- **URL**: https://www.notion.so/311b6d88d2cf8015a5f7cfe0d613931a

- **Dev Comment**: Root Cause: Timezone mismatch in date parsing. new Date('YYYY-MM-DD') parses as UTC midnight, which shifts to the previous day in UTC+9 (Korea). This caused countActiveForPatient() to query the wrong date and return 0 prescriptions.

Fix: Created DateUtil.parseDateString() utility that parses date strings as local midnight. Applied across all exercise controllers and services.

Files Changed: date.util.ts (new), exercise-day.controller.ts, exercise-log.controller.ts, exercise-prescription.controller.ts, exercise-day.service.ts, exercise-log.service.ts



## [2982] Delete notification icon

- **Status**: Close

- **URL**: https://www.notion.so/30fb6d88d2cf809fb4adf331323b242c

- **Dev Comment**: Removed the notification bell icon button from the /patient/survey page header. The button had no click handler and was non-functional. Cleaned up the unused Bell import from lucide-react.

Files modified:
- frontend/app/pages/patient/survey.tsx

Commit: fix: remove notification icon from patient survey page



## [2980] Adjust the video height

- **Status**: Close

- **URL**: https://www.notion.so/30fb6d88d2cf80f0a6acd48f185e36b0

- **Dev Comment**: Fixed exercise video layout to be full-width edge-to-edge. Removed padding, borders, and rounded corners from video container. Content below video retains proper padding. Container now uses flex-1 for proper height fill. Commit: 5e429d1



## [2978] Wrong Sender Label

- **Status**: Close

- **URL**: https://www.notion.so/30fb6d88d2cf8038a2fef6505e97b099

- **Dev Comment**: Root Cause: Patient chat-detail.tsx identified sender role using ID comparison only (senderId === coachId). When admin sends messages from dashboard and is also the coach for that room, messages appear as Coach instead of Admin.

Fix:
1. Backend: Added sender.role to Socket.IO broadcasts (admin.service.ts + chat.gateway.ts)
2. Backend: Fixed getRoom endpoint to load patient/coach relations
3. Frontend: Updated ChatMessage.sender type to include role field
4. Frontend: chat-detail.tsx now checks sender.role === 1 (ADMIN) before coachId comparison

Files: chat-detail.tsx, chat.d.ts, admin.service.ts, chat.gateway.ts, chat.controller.ts



## [2977] Message Sorting Issue

- **Status**: Close

- **URL**: https://www.notion.so/30fb6d88d2cf800a80a6e26227ccd554

- **Dev Comment**: Fixed: The backend chat message query (findByRoomId) was returning messages in DESC order (newest first) for pagination, but the frontend rendered the array as-is without reversing. This caused messages to appear newest-at-top instead of the standard oldest-at-top chat order.

Added .reverse() after the query so messages return in chronological order (oldest to newest). This fix applies to both patient and coach chat pages.

File changed: backend/src/modules/chat/repositories/chat-message.repository.ts



## [2976] Coach chat room is displayed as a seperated window

- **Status**: Close

- **URL**: https://www.notion.so/30db6d88d2cf8004a840d68934e1ad69

- **Dev Comment**: Removed redundant max-w-md mx-auto wrapper div from CoachChatDetail component. The coach layout already provides these container styles, so the double wrapping created a box-within-a-box effect with extra margins. Now uses React fragments like PatientChatDetail.



## [2975] Mismatch between the selected date and the clicking date.

- **Status**: Close

- **URL**: https://www.notion.so/30db6d88d2cf803095cdfd810cbf5081

- **Dev Comment**: Fixed timezone off-by-one bug in patient home calendar. handleDayClick used toISOString() which converts to UTC, shifting dates backward by 1 day in KST. Replaced with direct string formatting (YYYY-MM-DD). File: frontend/app/pages/patient/home.tsx (line 276-280)



## [2961] Dynamic Button Label based on Exercise Sequence

- **Status**: Close

- **URL**: https://www.notion.so/30cb6d88d2cf809a9442e6c185d577fb

- **Dev Comment**: Changed button text in exercise-detail.tsx to show "다음 운동" (Next Exercise) for first/middle exercises and "운동 종료" (Finish) for the last exercise. Uses existing isLastExercise variable. Navigation (next/previous) was already working correctly.



## [2960] Reduce the vertical space

- **Status**: Close

- **URL**: https://www.notion.so/30cb6d88d2cf803197e7eb3597306243



## [2951] Remove the hard-coding and display actual data

- **Status**: Close

- **URL**: https://www.notion.so/30cb6d88d2cf80ce9908e7d596105c92

- **Dev Comment**: Removed hardcoded empty exercises array from exercise history page. Expanded day cards now lazy-load actual prescription data via GET /exercise-prescriptions/my?date=. Each exercise shows title, duration, sets, completion status, and difficulty. Also fixed stale React Query key for month navigation.



## [2940] Failed to set a coach

- **Status**: Close

- **URL**: https://www.notion.so/309b6d88d2cf8046bbd8f57d30731a9e

- **Dev Comment**: Root cause: Patient detail page sends fullName, phone, birth fields but UpdateUserDto only accepted username, email, role, coachId. Global ValidationPipe with forbidNonWhitelisted:true rejected the request.

Fix:
- Added fullName, phone, birth to UpdateUserDto
- Updated AdminService.updateUser() to persist new fields
- Added duplicate coach assignment guard
- Updated frontend UpdateUserRequest type
- Fixed patient-detail.tsx mutation typing

Files: update-user.dto.ts, admin.service.ts, admin.d.ts, patient-detail.tsx



## [2921] New users’s Days elapsed is invalid (minus value)

- **Status**: Close

- **URL**: https://www.notion.so/306b6d88d2cf802e8e72ec7a4a54ab40

- **Dev Comment**: Fixed daysElapsed calculation — registrationDate was not zeroed to midnight like `now`, causing negative values for same-day users. Added `registrationDate.setHours(0, 0, 0, 0)` in both `mapUserWithAssignment()` and `getPatientsWithTodayPain()` in admin.service.ts.



## [2918] Reduce the vertical space

- **Status**: Close

- **URL**: https://www.notion.so/306b6d88d2cf800ea3baed9de9144652

- **Dev Comment**: Reduced vertical spacing on exercise detail page (/patient/exercise/:id). Changes: (1) Video changed from square to 16:9 aspect ratio, (2) Reduced all vertical margins and padding, (3) Removed mt-auto from navigation buttons so they sit directly below the timer instead of being pushed to the bottom. All elements now fit on one mobile screen without scrolling.



## [2917] Exercise timer should reset for new exercises but save/resume for previous ones

- **Status**: Close

- **URL**: https://www.notion.so/306b6d88d2cf8085b201f6ab970fedf0

- **Dev Comment**: Fixed per-exercise timer tracking in exercise-detail.tsx.

Changes:
1. Added useEffect on exercise ID change that resets timer to 00:00 for new exercises or restores saved time from DB logs for previously-completed exercises
2. Timer value (delta only) is now sent to backend via timeSpentSeconds in logExercise mutation
3. Backend accumulation logic was already correct - this was purely a frontend implementation gap

File modified: frontend/app/pages/patient/exercise-detail.tsx



## [2916] Remove the space in patient name

- **Status**: Close

- **URL**: https://www.notion.so/306b6d88d2cf80d9a897ea4a7abbd105

- **Dev Comment**: Fixed: Replaced firstName+lastName concatenation with fullName field in prescription-management.tsx. The page was manually joining firstName and lastName with a space, causing extra whitespace when one field was empty. Updated getPatientName(), getPrescribedByName(), and search filter to use fullName — consistent with all other dashboard pages.



## [2915] Expand selection exercise list 

- **Status**: Close

- **URL**: https://www.notion.so/306b6d88d2cf80bebfe8ca180756a19a

- **Dev Comment**: Fixed: Increased min-h from 150px to 300px on the selected exercises section in PrescriptionEditorPanel.tsx (line 137) to show 5 exercises by default. File: frontend-dashboard/app/components/prescriptions/PrescriptionEditorPanel.tsx



## [2914] Timeline dot-lined overlap event item

- **Status**: Close

- **URL**: https://www.notion.so/306b6d88d2cf806397a3cfdf0abbb918

- **Dev Comment**: Fixed in CalendarStyles.css:
1. Changed --fc-page-bg-color from transparent to var(--background) - restores 1px solid outline (box-shadow) around events that creates visible gaps between adjacent events in weekly/day views
2. Added solid background-color: var(--card) to .fc-timegrid-event - prevents dotted grid lines from showing through events
3. Fixed all hsl(var(--...)) patterns to var(--...) for Tailwind CSS v4 compatibility (oklch color values were being wrapped in hsl() which is invalid)



## [2904] keep only "Detailed" view

- **Status**: Close

- **URL**: https://www.notion.so/303b6d88d2cf8093b674ced13e0e9a91



## [2900] Improve Calendar event layout and Fix line visibility

- **Status**: Close

- **URL**: https://www.notion.so/303b6d88d2cf8088bd49f36c8548cbcd



## [2899] User roles should only be COACH, PATIENT, ADMIN

- **Status**: Close

- **URL**: https://www.notion.so/303b6d88d2cf8025b419ee8e7a11ca3b



## [2898] Chat room should be 3-person (patient + coach + admin), not 2-person

- **Status**: Close

- **URL**: https://www.notion.so/303b6d88d2cf805b9eeef0a2ded828f5



## [2360] Failed to create a new user

- **Status**: Close

- **URL**: https://www.notion.so/302b6d88d2cf806ea8ccc5aafea18548



## [2359] Prescription record(history)

- **Status**: Close

- **URL**: https://www.notion.so/301b6d88d2cf8040b782cb54e3a6d71c



## [2358] Exercise library

- **Status**: Close

- **URL**: https://www.notion.so/301b6d88d2cf80d8a1dbdf20d80dec21

- **Dev Comment**: Root cause: The seed script populated rep_time and unit columns but never populated default_reps and default_duration_seconds. The frontend reads defaultReps/defaultDurationSeconds to display reps/times, so all exercises showed "-". Additionally, exercise 8102 had Unit=분 (minutes) which the seed script did not handle, resulting in NULL unit.

Fix:
1. Migration 1770300000000: Fixes exercise 8102 (1분 → 60초), populates default_reps from rep_time for REPS exercises, populates default_duration_seconds from rep_time for SECONDS exercises.
2. Seed script updated: Added 분/minutes unit handling with seconds conversion. Added default_reps and default_duration_seconds to INSERT statement.



## [2338] User management

- **Status**: Close

- **URL**: https://www.notion.so/2feb6d88d2cf80af9fe3c33a9d77516c



## [2337] Patient has two chatroom with the same coach

- **Status**: Close

- **URL**: https://www.notion.so/2feb6d88d2cf8006a5f5f9e2203fcba4

- **Dev Comment**: ### Fix Applied (Backend)

**Issue 1: Duplicate chat rooms**
- Controller `POST /chat/rooms` now uses `getOrCreate` pattern instead of `create` to prevent duplicates
- Added race condition protection to `getOrCreate()` service method (catches PostgreSQL unique constraint violation 23505)
- Created migration `1770100000000-CleanupDuplicateChatRooms` to merge existing duplicate rooms, move messages, and replace UNIQUE constraint with partial unique index

**Issue 2: Last message not showing in chat list**
- Added `getLastMessagesByRoomIds()` batch method to ChatMessageRepository using PostgreSQL DISTINCT ON
- Updated `findWithUnreadCount()` in ChatRoomService to fetch and attach last message for each room
- Frontend already handles `room.lastMessage` correctly - no frontend changes needed

**Files Modified:**
- `backend/src/modules/chat/repositories/chat-message.repository.ts`
- `backend/src/modules/chat/services/chat-room.service.ts`
- `backend/src/modules/chat/controllers/chat.controller.ts`
- `backend/src/database/migrations/1770100000000-CleanupDuplicateChatRooms.ts` (new)



## [2335] Patient can not see today’s exercise

- **Status**: Close

- **URL**: https://www.notion.so/2feb6d88d2cf800bab4ff8241e5d151c



## [2334] Patient can not submit their survey

- **Status**: Close

- **URL**: https://www.notion.so/2feb6d88d2cf80ea8c0aef074e334fea



## [2332] Coach profile details

- **Status**: Close

- **URL**: https://www.notion.so/2feb6d88d2cf80e990f7faee92846ce9

- **Dev Comment**: Added min-h-0 to the profile content wrapper div to fix flex overflow scrolling. The layout uses min-h-screen + overflow-hidden + flex-col, so without explicit min-h-0, the flex item couldn't shrink below content height and overflow-y-auto never activated. Also added no-scrollbar, z-10, and relative to match the pattern used by other working coach pages (patients, calendar, chat). Applied same fix to patient profile for consistency.



## [2333] User are unable to select another date

- **Status**: Close

- **URL**: https://www.notion.so/2feb6d88d2cf8058b92adaeb7edc6bcd

- **Dev Comment**: Fixed: Made patient home page (/patient) dynamic with date selection.

Changes:
1. Calendar dates are now clickable (week view + expanded). Selecting a date updates all content (survey, exercises, coaching session) for that date.
2. Added selected date visual indicator: blue ring in both views (distinct from today solid blue).
3. Week view now shows colored dots for all activities: green=exercise, yellow=survey, blue=session.
4. Fold button icon changed to proper upward arrow.
5. Date-aware labels adapt based on selected date. Past dates hide survey Start button.

Files: shared.d.ts, useSurveys.ts, calendar-grid.tsx, home.tsx
No backend changes.



## [2326] Need to prescriptions list page

- **Status**: Close

- **URL**: https://www.notion.so/2feb6d88d2cf80368684ea8835fb2448



## [2222] Exercise creation feature

- **Status**: Close

- **URL**: https://www.notion.so/2feb6d88d2cf80609dbed81dae604fa9



## [1530] Use can see the history of submitted survey

- **Status**: Close

- **URL**: https://www.notion.so/cf79d40b50c9448e9538ec29b20ba485

- **Dev Comment**: Fixed: Survey history was limited to 10 entries with no pagination. Updated useSurveyHistory hook to support pagination parameters and return full response (surveys, total, page, limit). Added accumulating Load More button to patient survey history tab so users can view their complete survey history. The existing detail modal (click to view survey details) was already working correctly.



## [1531] admin log in failed message

- **Status**: Close

- **URL**: https://www.notion.so/ca960c93a68e40b280cc5ac9a075a537

- **Description**: There is no message for an invalid password.



## [1526] minus number

- **Status**: Close

- **URL**: https://www.notion.so/02b0388b606148f3ba8bf47f45f32eaa

- **Description**: All number input fields accept positive integers from 0.



## [1525] Exercise prescription

- **Status**: Close

- **URL**: https://www.notion.so/36dd35eece54474e8e00b2f26c872a1c

- **Description**: The admin can prescribe the patient with 3-7 exercises. They need the search and filter exercise feature when prescribing.



## [1519] Admin profile avatar

- **Status**: Close

- **URL**: https://www.notion.so/ffd52fcae2db489f92e0fb112e30016a

- **Description**: A logout dropdown menu is required.



## [1517] Notification Icon

- **Status**: Close

- **URL**: https://www.notion.so/d3b879167c2844a98b85f679975f1ec4

- **Description**: I don't think admin need the notification detail page



## [1516] Too many Role

- **Status**: Close

- **URL**: https://www.notion.so/5cd31945fc19406b8c870b0c5d466307

- **Description**: The app has only one admin and two users (patient, coach)



## [1524] User creation

- **Status**: Close

- **URL**: https://www.notion.so/122fb6029a7941c39302438d2dbfc8e4

- **Description**: 1. Username 
2. login ID 
3. Password 
4. Password confirm 
5. Birth
6. phone *
7. Role



## [1515] Localization Issue on Login Button

- **Status**: Close

- **URL**: https://www.notion.so/78f944b1beac465d9588e9d6a68e8bb4

- **Description**: For a brief moment, it appears in English rather than Korean.

- **Dev Comment**: Fixed: Added missing i18n key "auth.landing.loggingIn" to both ko.json ("로그인 중...") and en.json ("Logging in..."). The key was referenced in login.tsx but never defined in the translation files, causing i18next to display the raw key string instead of the translated text.



## [1508] All the component sizes

- **Status**: Close

- **URL**: https://www.notion.so/752a7708e68f479b925c039fe1dd1e46

- **Description**: The header heights and profile size are different eachother. We need to bring all the sizes together. and don't need the red section if there is no exercise presciption



## [1507] Calendar gap(height) between the elements

- **Status**: Close

- **URL**: https://www.notion.so/bd7eb0af591d4974acf459c519657136

- **Description**: It is too large.



## [1513] Coach profile page

- **Status**: Close

- **URL**: https://www.notion.so/f088602e106348a08326a54d565641f3

- **Description**: There is no coach profile page now. Coach can't log out.



## [1506] My profile detail page

- **Status**: Close

- **URL**: https://www.notion.so/464be8b04c8d4a2c81d6f63865fa3ef2

- **Description**: It should be directed to my profile page, not a dropdown.



## [1509] Padding sizes

- **Status**: Close

- **URL**: https://www.notion.so/69d36a23f0b84a0a81a95817c866d1e0

- **Description**: Each has different padding.



## [1512] Quality ranges

- **Status**: Close

- **URL**: https://www.notion.so/2c47f5bfb14e475eac1057e59a276ab2

- **Description**: The sleep quality assessment scale ranges from 1 to 10.



## [1510] Survey check effect

- **Status**: Close

- **URL**: https://www.notion.so/8c1982e02ad547b098fb84ac1ae7aed5

- **Description**: If the user chooses an option, the circle will be filled with dot.



## [3015] show password not matching in create account, confirm password as instruction or warning

- **Status**: Ready for test

- **URL**: https://www.notion.so/313b6d88d2cf801983e6c0b677acd164



## [3014] Onboarding information complete issue, loading in username

- **Status**: Ready for test

- **URL**: https://www.notion.so/313b6d88d2cf80348a3bed5926ee9b64



## [3013] SEARCH not connected with db

- **Status**: Ready for test

- **URL**: https://www.notion.so/313b6d88d2cf8076aa58db1da927cb36



## [3012] No image showing in identify artwork.

- **Status**: Ready for test

- **URL**: https://www.notion.so/313b6d88d2cf8041869bedd2f7178e09



## [3011] Object count issue in recognition picture

- **Status**: Ready for test

- **URL**: https://www.notion.so/313b6d88d2cf8062bcb7c91566dfe4c6



## [3010] No images showing on detected image. 

- **Status**: Ready for test

- **URL**: https://www.notion.so/313b6d88d2cf80c4aa48c65227c39600



## [3009] Images ratio adjustment must be corrected. It’s bad now.

- **Status**: Ready for test

- **URL**: https://www.notion.so/313b6d88d2cf804eac3ec6ef73d02a44



## [3008] Capture collection not created

- **Status**: Ready for test

- **URL**: https://www.notion.so/313b6d88d2cf806e86b9cf31758e6a01



## [3007] Password reset not works

- **Status**: Ready for test

- **URL**: https://www.notion.so/313b6d88d2cf80749ed3d223a25f469c



## [3006] Button text overlay issue

- **Status**: Ready for test

- **URL**: https://www.notion.so/313b6d88d2cf80978dc2ef5842e49f19



## [2957] 
specs for the design elements used?i.e.) Typography Hierarchy, etc Font Size / Color RGB 

- **Status**: New

- **URL**: https://www.notion.so/30cb6d88d2cf806195f4dab56eaf0e02



## [3133] Clickable button on Order Management & Add “Request Revision” “Awaiting Production” button instead of “On hold” and “Cancelled”

- **Status**: New

- **URL**: https://www.notion.so/320b6d88d2cf80c684a0d768daa21cf8



## [3132] Add more buttons from Dashboard as admin review below most frequently
“Drafted”, “Request Revision”, “Awaiting Production”

- **Status**: New

- **URL**: https://www.notion.so/320b6d88d2cf8027823dcc0fa5c94769



## [3130] Design review 3shape link insert and html insert design to be same.

- **Status**: New

- **URL**: https://www.notion.so/320b6d88d2cf80e98c1ae275f0ec0444

- **Description**: 6



## [3129] When Reopen Order, “User ID” & and “Case Name” should be already inserted.

- **Status**: New

- **URL**: https://www.notion.so/320b6d88d2cf80d0a4e4c3cddccf9a7c



## [3128] Growth to be Last 30 days, not month-by-month

- **Status**: New

- **URL**: https://www.notion.so/320b6d88d2cf80c28951dcd482505052



## [3127] Dashboard update to daily x-axis

- **Status**: New

- **URL**: https://www.notion.so/320b6d88d2cf80a38cedf89c7a98716a



## [3126] Need “Scanbody Type” section in Manage Content -> Order items (admin need to add more items)

- **Status**: New

- **URL**: https://www.notion.so/320b6d88d2cf80a49111fde76219135b



## [3125] Add “ASC Abutment” in Abutment Type, and “Screwmentable” Retention Type. ASC Abutment is $150.

- **Status**: New

- **URL**: https://www.notion.so/320b6d88d2cf808884aedec1c395e4d2



## [3124] Add “Veneer” in Restoration type, same decision tree as “Single Crown” → automatically add “Printed Model” in Additional Services

- **Status**: New

- **URL**: https://www.notion.so/320b6d88d2cf80838234e3e0e7b0bb6d

- **Description**: 4.1



## [3123] Added item not displayed in Order summary

- **Status**: New

- **URL**: https://www.notion.so/320b6d88d2cf80f99dfed9220e4e01f3

- **Description**: 1.2



## [3122] Added item not displayed in invoice

- **Status**: New

- **URL**: https://www.notion.so/320b6d88d2cf80899301cae74de0e089

- **Description**: 1.1



## [3121] Current chat functionally unstable

- **Status**: New

- **URL**: https://www.notion.so/320b6d88d2cf800eb5c1f797e36edce1

- **Description**:   • Does not display text after clicking box
  • Download button becomes smaller for a long file name
  • Text displays random chat area when clicked → should display the latest conversation
  • wrong chat display (Dr. Erin’s chat is Dr. Mark Valrose’s case)

[After several attempts, it displays the correct chat.]



## [3110] Page Reload Redirects to First Page

- **Status**: Close

- **URL**: https://www.notion.so/31eb6d88d2cf804faba3e7276daa44c5

- **Description**: No matter which page I am currently on, if I reload that page, it always redirects me to the first page instead of staying on the current page.



## [3109] Image Generation Issue

- **Status**: New

- **URL**: https://www.notion.so/31eb6d88d2cf80a8af5ed8cf345618db



## [3108] Error When Deleting Prompt After Design Generation (“Validation failed – uuid is expected”)

- **Status**: New

- **URL**: https://www.notion.so/31eb6d88d2cf804ea32ad28b62d62b5f

- **Dev Comment**: Added DELETE /design-editing/messages/:messageId endpoint with HTML rollback. Backend deletes user msg + assistant response + all subsequent msgs, rolls back page HTML to previous snapshot. Frontend trash icon on edit messages with confirmation dialog. Files: design-editing.service.ts, design-editing.controller.ts, designEditingService.ts, chatGenerationSlice.ts, UnifiedMessageList.tsx, chat-generate/index.tsx



## [3107] Generated Landing Page Appears Blank in Design View

- **Status**: New

- **URL**: https://www.notion.so/31eb6d88d2cf80f2b553d3b76e7e33cf

- **Dev Comment**: Fixed: 1) Applied sanitizeInlineScripts to ALL iframe HTML (both preview and design modes) to prevent broken </script> tags in AI-generated HTML from corrupting the DOM and hiding page content. 2) Increased backend edit timeout from 80s to 120s to reduce timeout errors on large pages.



## [3106] Side Navbar issues among pages

- **Status**: New

- **URL**: https://www.notion.so/31eb6d88d2cf8084a8c8d5a36a08b821



## [3105] Generates Blank pages

- **Status**: New

- **URL**: https://www.notion.so/31eb6d88d2cf808e9c68c68099acc4af

- **Dev Comment**: Added global edit detection: when user mentions all pages/every page/globally etc in edit prompt, the backend now applies the same AI edit to all sibling pages automatically. Frontend re-fetches design pages after global edit to reflect changes.



## [3104] Navbar Should Be Hidden in Design Workspace

- **Status**: Close

- **URL**: https://www.notion.so/31db6d88d2cf80a48067f46d5eef72ad

- **Dev Comment**: Fixed: Removed the app Header from the chat-generate workspace layout. The workspace already has its own back button and toolbar, giving users a distraction-free full-screen design environment.



## [3103] Prompt-Based Minor Edits Not Working (Edit Stream Connection Lost)

- **Status**: Close

- **URL**: https://www.notion.so/31db6d88d2cf8015bf97ea0ea42114f9



## [3102] Page Organization and Prototype Navigation Problems

- **Status**: New

- **URL**: https://www.notion.so/31db6d88d2cf806194eef1b3d0426408

- **Dev Comment**: Fixed 3 remaining issues: (1) Auto-group now triggers when restoring existing sessions, not just after fresh generation. (2) Anchor links (#section, #features, etc.) no longer blocked � properly skipped for in-page scrolling. (3) AI-generated links with spaces (e.g. "About Us") now normalized to hyphens before slug matching. Also improved: query string handling in catch-all regex, consistent lowercase slug normalization.



## [3100] Blank Page After Design Generation

- **Status**: Close

- **URL**: https://www.notion.so/31bb6d88d2cf807cbcfdc1fc365d1150



## [3099] Mobile App Screen Not Using Full Width (Responsiveness Issue

- **Status**: Close

- **URL**: https://www.notion.so/31bb6d88d2cf8009922debab43f6aa43



## [3098] Default View Should Open in Preview Mode Instead of Edit Mode

- **Status**: Close

- **URL**: https://www.notion.so/31ab6d88d2cf8004bd3ec2685be43e36

- **Dev Comment**: Added Preview/Design view mode toggle in chat-generate page. Default is now Preview mode (no element selection). Users can switch to Design mode to click-edit elements. Also applied to my-designs detail page.



## [3097] Preview Container Not Optimized for 1440px Layout

- **Status**: Close

- **URL**: https://www.notion.so/31ab6d88d2cf80b0a731cde07911fde2

- **Dev Comment**: Auto-fit zoom: preview now calculates optimal zoom based on container width to fill available space. Reduced padding from p-4 to p-2 in desktop mode. Removed hardcoded scale(0.6) from my-designs and GeneratingView, using full-width iframes instead.



## [3095] Inner Page Preview Not Working

- **Status**: Close

- **URL**: https://www.notion.so/31ab6d88d2cf808b8724c049ff69e5dc

- **Dev Comment**: Preview button always opened /designs/{id}/preview (no slug), showing the home page regardless of selected page. Fix: append current page slug to preview URL so inner pages open correctly.



## [3094] History Does Not Show Any Updates made by prompts

- **Status**: Close

- **URL**: https://www.notion.so/31ab6d88d2cf8083ba06ca3f77878368

- **Dev Comment**: Version history was client-side only (Redux state), lost on page reload. Fix: in fetchEditMessagesByPage.fulfilled, rebuild version history from persisted htmlSnapshot fields in edit messages. Edit history now survives page refresh.



## [3093] Connection Error During Repeated Matching

- **Status**: Ready for test

- **URL**: https://www.notion.so/31ab6d88d2cf805d887dd7963f9397f9



## [3092] An unknown component appears 

- **Status**: Ready for test

- **URL**: https://www.notion.so/31ab6d88d2cf808682b5c71913972de4



## [3091] Inner Pages Generated Without Design

- **Status**: Close

- **URL**: https://www.notion.so/31ab6d88d2cf80b19feecd098f31c2b6

- **Dev Comment**: Code flow is correct (commit 3be832b fixed template-free sessions). Added retry mechanism: if generated page body text < 200 chars (empty/skeleton), regenerates once. This catches cases where AI produces minimal HTML.



## [3090] Bug: Captured artworks are not being sorted into the correct collection based on check-in status.
Expected: If user is checked in, capture should be added to the Location Capture Collection; if not checked in, capture should be added to the Untitled Capture Collection.

- **Status**: Ready for test

- **URL**: https://www.notion.so/31ab6d88d2cf80fa9affe5cc75e4d84a



## [3089] Bug: When user captures an artwork photo, the Artwork Inference API is not being called, resulting in no AI recognition functionality during the Sprint 2 demo.
Expected: Artwork Inference API should be called immediately after photo capture, regardless of check-in status, and return artwork information on success or prompt manual entry on failure.

- **Status**: New

- **URL**: https://www.notion.so/31ab6d88d2cf803986fcfff7e308528e



## [3082] Optimized Social Login Flow

- **Status**: In Progress

- **URL**: https://www.notion.so/319b6d88d2cf80d38e20f3604e916992



## [3081] Attached Image Not Displaying in Chat

- **Status**: Close

- **URL**: https://www.notion.so/319b6d88d2cf8019b58ad227e592d5c6

- **Dev Comment**: Images were sent to backend but never displayed in chat UI. Fix: added imageDataUrl to ChatMessage type, passed attachment data URL through Redux, and rendered image preview in MessageBubble component. Note: images show in current session only (not persisted to DB).



## [3051] Incorrect Output from PDF Analysis

- **Status**: Close

- **URL**: https://www.notion.so/318b6d88d2cf80bdb518dfccb7358f42

- **Dev Comment**: Root cause: analyzeWithPrd() ignored uploaded PDF/image, determining pages from text only. A single landing page PDF was treated as a broad request, generating 20+ unrelated pages. Fix: pass imageData to PRD analysis + updated prompt to respect visual references.



## [3047] Chat History Not Fully Displayed

- **Status**: Close

- **URL**: https://www.notion.so/317b6d88d2cf80139cd5e7d897921520

- **Description**: All we can see is our first prompt. when we reload the website no prompt showing except the first prompt

- **Dev Comment**: Additional fixes: (1) Added isRestoringSession loading state � shows skeleton loading UI while chat history is being fetched on page reload instead of the welcome prompt. (2) Generation complete status is derived from session state on restore (showDerivedCompleteMessage). (3) Messages are sorted chronologically after fetch. Previous fix already handles AI response display and FILE: marker stripping.



## [3046] Same template new page creation error

- **Status**: Close

- **URL**: https://www.notion.so/317b6d88d2cf800781c3e570f91d9d64

- **Description**: When we try to create new inner page its not creating, showing “Failed to add page”

- **Dev Comment**: Root cause: addPageToDesign required templateId, but template-free chat sessions have null templateId. Fix: use first existing page HTML as style reference when no template. Also fixed error message priority (backend message > generic Axios error).



## [3045] Second Prompt Not Being Processed

- **Status**: Close

- **URL**: https://www.notion.so/317b6d88d2cf8001b6f9c73093e68f33

- **Description**: When I give the second or another prompt in the chat box, the second prompt doesn't appear in the chat section but it shows starting design generation but don’t do the design. 

- **Dev Comment**: Root cause (regression): The addPagePattern regex was too broad � it matched any prompt containing make/create/generate + any text + page. So edit prompts like "make this page look better" falsely triggered addNewPage instead of the edit path. Combined with UnifiedMessageList only rendering messages before gen-complete, user messages were invisible. Fix: (1) Tightened regex to require action word near page, (2) render post-gen messages, (3) backend uses BadRequestException, (4) await sendChatMessage before SSE, (5) guard for completed sessions without designId.



## [3044] Design Rendering Shows Raw Code

- **Status**: New

- **URL**: https://www.notion.so/317b6d88d2cf8023af87d04b41119d3f

- **Description**: Some of the page is displaying raw code instead of rendering the proper design.

- **Dev Comment**: Root cause: AI-generated inline <script> tags contain literal </script> strings that break the browser parser, causing subsequent injected scripts to render as visible text. Fix: Added sanitizeInlineScripts() on both frontend (iframe-selection-script.ts) and backend (chat-generation.service.ts, gemini.service.ts) to escape nested </script> within script blocks. Also ensured missing </body></html> closing tags are appended.



## [3043] Version History Not Restoring Previous Design

- **Status**: Close

- **URL**: https://www.notion.so/317b6d88d2cf8012b0eed9ad5c083adc

- **Dev Comment**: Two fixes: 1) Version preview was wrapping full HTML docs inside <body>, creating malformed nested HTML. Now detects full docs and uses them directly. 2) Version restore now persists to backend via quickEdit API so it survives page refresh.



## [3042] Design Disappeared After Manual Text Change

- **Status**: Close

- **URL**: https://www.notion.so/317b6d88d2cf80fcb9cec8e01dad1dcb

- **Dev Comment**: Root cause: DOMParser re-serialization corrupts complex generated HTML. Fix: capture full page HTML directly from iframe DOM after text edit (removing overlay elements temporarily). Falls back to DOMParser path if iframe HTML unavailable.



## [3041] suggested workflow is:

- **Status**: Backlog

- **URL**: https://www.notion.so/317b6d88d2cf80398668c46e91f322c0

- **Dev Comment**: Implemented all 3 features: (1) Template selector dialog with search and live preview in chatbox (Layout icon button, shown in chatting phase). (2) Attachment upload already existed (Paperclip button). (3) Image clipboard pasting - paste images directly into chatbox via Ctrl+V.



## [3034] Incomplete Design Output from Template for first time

- **Status**: Close

- **URL**: https://www.notion.so/317b6d88d2cf804c8ba7e48cd6f433e7

- **Description**: When we try to generate using template initially it doesn’t provide any design. But when we reload the page we can see the design properly.

- **Dev Comment**: Fixed missing retry logic for fetchDesignBySession after generation completes. The chat SSE handler had retry logic but generation SSE, fallback polling, and safety net polling did not. Extracted shared fetchDesignWithRetry callback with exponential backoff (up to 3 attempts) and applied it to all 4 generation completion handlers. Design now loads reliably after generation without requiring a page reload.



## [3020] No Download HTML option after generationScreenshot

- **Status**: Close

- **URL**: https://www.notion.so/313b6d88d2cf81fcbee8cec5aa20ec41

- **Description**: After a design is generated and shows 100% completion, there is no option available to Download the HTML code.

- **Dev Comment**: Download HTML/React buttons were gated behind isMultiPage condition. Removed the guard so buttons appear for all designs once a designId exists.



## [3019] template_id null constraint error when saving design

- **Status**: Close

- **URL**: https://www.notion.so/313b6d88d2cf813782a6ef4ee5e4aacf



## [3018] Generated sections not rendering in design preview

- **Status**: Close

- **URL**: https://www.notion.so/313b6d88d2cf81d69d7ad318642e0acc



## [1547] [High] Review design button inactive on request revision status

- **Status**: Ready for test

- **URL**: https://www.notion.so/2f7b6d88d2cf81ffa931c0f62ab7b07b

- **Description**: Customers can't easily check for new designs.

Tag: @Zihad



## [1546] [Urgent] Revision request status not visible in dashboard

- **Status**: Ready for test

- **URL**: https://www.notion.so/2f7b6d88d2cf8157a4b1f4b27cfca90a

- **Description**: Status not seen unless checking notification bar.

Tag: @Zihad



## [1545] [Ivory] Review design button inactive on request revision status

- **Status**: Ready for test

- **URL**: https://www.notion.so/2f7b6d88d2cf81a88359dfc6c8b63b8a

- **Description**: [Medium urgency] "Review design" button is inactive when status is on request revision. Customers need to consistently check for new designs unless notified via email.



## [1544] [Ivory] Revision request status not visible in dashboard

- **Status**: Ready for test

- **URL**: https://www.notion.so/2f7b6d88d2cf81da8640d20bebcb51f2

- **Description**: [Immediate attention needed] Revision request status not seen in the dashboard, easy to miss unless we check the notification bar. See attached images in Slack.



## [1543] Referral codes not following exm000 format for early users

- **Status**: Close

- **URL**: https://www.notion.so/2f7b6d88d2cf810b9a5bfdabd21d1bdf

- **Description**: Referral codes do not follow the agreed "exm000" format. Users with ID up to US00018 have referral codes that do not start with "exm".

Expected: All referral codes should follow exm + number format (e.g., exm001, exm045)
Actual: Early users (US00001-US00018) have different format

Reported by: Sun



## [1542] Orders not shown in User Details - Order Information table missing orders

- **Status**: Close

- **URL**: https://www.notion.so/2f7b6d88d2cf8121b433d0672f3fdcdc

- **Description**: Some orders do not appear in the Order Information section under User Details. This has been noticed with multiple user accounts.

Example: User US00045 (Dr. Patrick Loftus) shows "2 Total Orders" and "2 Monthly Orders" but only 1 order visible in table. Could be pagination/filter issue, Confirmed vs Draft tab issue, or data mismatch.

Reported by: Sun



## [1365] Chat alert issue

- **Status**: Ready for test

- **URL**: https://www.notion.so/2b6b6d88d2cf80e68df3d8a6f19b7672

- **Description**: Not showing alerts on chat icon



## [1364] Add discount with $0

- **Status**: Ready for test

- **URL**: https://www.notion.so/2b6b6d88d2cf80d5a811c9b07aa0f526

- **Description**: Change range for 0 to be valide input



## [1363] Design review comment notification

- **Status**: Ready for test

- **URL**: https://www.notion.so/2b6b6d88d2cf80799591c58ca5ee8450

- **Description**: No notification or alerts, marked as read in Messages tab in Admin portal



## [1362] Not syncing on live server

- **Status**: Close

- **URL**: https://www.notion.so/2b2b6d88d2cf80bd903bcc8094728be3

- **Description**: status and notification not live synced           



## [1361] Merge order does not work

- **Status**: Ready for test

- **URL**: https://www.notion.so/2b2b6d88d2cf80ae8b27e38cb54c9b12

- **Description**: Production site



## [1346] Referral code issue

- **Status**: Ready for test

- **URL**: https://www.notion.so/2aab6d88d2cf804db267d4906e509d0c

- **Description**: Cannot skip & submit or add referral code. Gives invalid phone number error



## [1345] Edit order issue

- **Status**: Close

- **URL**: https://www.notion.so/2aab6d88d2cf80e583b1ea69490e26ad

- **Description**: Duplicative Abutment Design fields in one page



## [1344] Status update for Refund request on Client end

- **Status**: Close

- **URL**: https://www.notion.so/2aab6d88d2cf809ab357e0662ea9fa3e

- **Description**: Status update on Track this order

Client requested refund —> “Refund requested”

Admin refunded —> “Refunded”

Refund rejected —> “Refund declined”



## [1343] Refund item in order

- **Status**: Close

- **URL**: https://www.notion.so/2aab6d88d2cf80d3aa27c26b4c7a22eb



## [1342] Tooth numbering issue

- **Status**: Close

- **URL**: https://www.notion.so/2aab6d88d2cf80fcb93fd810d88b89dd



## [1341] Missing abutment calculation in Implant full arch

- **Status**: Close

- **URL**: https://www.notion.so/2aab6d88d2cf8020a42bf603faeb1d6d



## [1340] No need for Interproximal contact in Implant full arch

- **Status**: Close

- **URL**: https://www.notion.so/2aab6d88d2cf803683cce3883fe1075d



## [1339] Glass ceramic should not be option in full arch

- **Status**: Close

- **URL**: https://www.notion.so/2aab6d88d2cf8022a5b7e4e3c61dfa76

- **Description**: Only 4 options available, Monolithic zirconia, High translucent zirconia, Multi-layer zirconia, PMMA Temporary (or Temporary) 



## [1338] Back button & changing input doesn’t affect price estimate

- **Status**: Close

- **URL**: https://www.notion.so/2aab6d88d2cf8055942df124198c4812



## [1336] Status issue

- **Status**: Close

- **URL**: https://www.notion.so/2aab6d88d2cf8052a246d8495f772e56

- **Description**: All changes to Awaiting production after payment by client



## [1335] Calendly issue

- **Status**: Close

- **URL**: https://www.notion.so/2aab6d88d2cf80f39ca1efca9e672a77

- **Description**: No issues. might be server error



## [1334] New address not functioning

- **Status**: Close

- **URL**: https://www.notion.so/2aab6d88d2cf806cafc9c463db007bac

- **Description**: Does not show after adding a new addresss 

it’s working



## [1333] Leads to wrong page

- **Status**: Close

- **URL**: https://www.notion.so/2aab6d88d2cf8057a99bf31a30b5eb9f

- **Description**: Credit memo and invoice in the order detail do not match



## [1332] Continue button inactivation error during “Edit order”

- **Status**: Close

- **URL**: https://www.notion.so/2aab6d88d2cf80eca45bdebced726a63



## [1331] New function “Save order” in Dashboard button

- **Status**: Close

- **URL**: https://www.notion.so/2aab6d88d2cf80eb94d1ea17c054b817



## [1330] Price display after removing item during Edit order

- **Status**: Close

- **URL**: https://www.notion.so/2aab6d88d2cf800f9473eafdfac5811b



## [1329] PMMA description shows without toogle on

- **Status**: Close

- **URL**: https://www.notion.so/2aab6d88d2cf80d7908ef25c6f983700



## [1328] Pontic not displayed in Order Details

- **Status**: Close

- **URL**: https://www.notion.so/2aab6d88d2cf809db6b4fa080b2f277f



## [1327] Dropdown still gets cut out

- **Status**: Close

- **URL**: https://www.notion.so/2aab6d88d2cf80ff82c8d63fab4778c9



## [1326] Viewable button not working

- **Status**: Close

- **URL**: https://www.notion.so/2aab6d88d2cf800e8f40fcbb5f14bbdc



## [1325] After refunded by admin, should have “Refunded” status

- **Status**: Close

- **URL**: https://www.notion.so/2a9b6d88d2cf8073befdd9c11ce9d14f



## [1324] Back button need to go to “Order” page

- **Status**: Close

- **URL**: https://www.notion.so/2a9b6d88d2cf80f6952bdfe0fc304266

- **Description**: Back button currently goes to where ever you came from, but should go to Order page as default.



## [1323] Order page loops back

- **Status**: Close

- **URL**: https://www.notion.so/2a9b6d88d2cf80edb66be798fdd1fa3a

- **Description**: When Order Edit & Implant Crown, order page Abutment Design loops back to Abutment Details so I have to go through the same two pages again. It loops once and it moves forward to Crown Details.

Abutment Design —> Abutment Details —> Abutment Design —> Abutment Details —> Crown Details…



## [1321] Proceed to Production button static

- **Status**: Close

- **URL**: https://www.notion.so/2a9b6d88d2cf8043ad47fbf9f7cc65b3



## [1304] Fp3 selected but no pink tissue showed

- **Status**: Close

- **URL**: https://www.notion.so/2a8b6d88d2cf80a9884cff501ed19ee7



## [1303] Toggle off but not showing fp1, fp2, fp3. only fp3 showing.

- **Status**: Close

- **URL**: https://www.notion.so/2a8b6d88d2cf802ebcf0f30c7284f6c8



## [1302] No retention type for full arch

- **Status**: Close

- **URL**: https://www.notion.so/2a8b6d88d2cf8068b1f6ff65127661b9

- **Description**: for
Design Arch & provide abutments, Design and Mill Arch & provide abutments
show only Scanbody Type, MUA Abutment Type, Implant Type, Implant Connection



## [1301] Design Arch Only

Design Arch & Mill

Design Arch & Print


- **Status**: Close

- **URL**: https://www.notion.so/2a8b6d88d2cf80eca40ec37b36e8e7df

- **Description**: Only MUA user, MUA scanbody type selection will be here for Design Arch Only

Design Arch & Mill

Design Arch & Print




## [1300] MUA used, MUA scanbody type shouldn’t be display when 1. Design Arch & provide abutments
2. Design and Mill Arch & provide abutments selected

- **Status**: Close

- **URL**: https://www.notion.so/2a8b6d88d2cf8066a8d7d3b63f928d15

- **Description**: MUA used, MUA scanbody type shouldn’t be display when 1. Design Arch & provide abutments
2. Design and Mill Arch & provide abutments selected

Condition wrong - wrong options



## [1297] Admin can’t edit full arch draft

- **Status**: Close

- **URL**: https://www.notion.so/2a8b6d88d2cf80069f28fad5a157ece7



## [1295] Custom margin

- **Status**: Close

- **URL**: https://www.notion.so/2a8b6d88d2cf80a6a425eeb29a04accb

- **Description**: Buccal - “Others - Please specify in Additional Instructions” - missing option

Lingual, Mesial, Distal  - missing after (-0.4)



## [1283] New registered login issue

- **Status**: Close

- **URL**: https://www.notion.so/2a7b6d88d2cf80948e5ed07b6d0f3c7d

- **Description**: Login goes to Registration, even when logo selected, still goes to Registration. Feels like loop created



## [1282] Edit bridge - success after select all tooth as crown but not continue after crown design

- **Status**: Close

- **URL**: https://www.notion.so/2a7b6d88d2cf803eaec3e28190d30f48

- **Description**: Order submitted - range 3 - crown 2 -pontic 1. 
Edit - select all 3 crown. - 0  pontic. 
- not continue after crown design while editing. but information updated. 



## [1281] Pink tissue - original

- **Status**: Close

- **URL**: https://www.notion.so/2a7b6d88d2cf80a4b36bd07ded647f11

- **Description**: Pink tissue option - Original - missing



## [1280] Gingival shade option missing

- **Status**: Close

- **URL**: https://www.notion.so/2a7b6d88d2cf8098b0fdca3e611f46df

- **Description**: c4, d2, d3, d4 not showing for Gingival shade



## [1279] Upload stl file - delete not working

- **Status**: Close

- **URL**: https://www.notion.so/2a7b6d88d2cf80c4956ae4ac5e2a6d77

- **Description**: After uploading, when delete STL file, skip for now doesn’t work. need to reupload stl to continue. 
For edit, when delete 



## [1278] Admin chat

- **Status**: Close

- **URL**: https://www.notion.so/2a7b6d88d2cf806e9158e4b0982e3567

- **Description**: - Showing Html tag element in chat preview admin 
- Link not clickable



## [1277] both bridge and implant bridge not showing crown materials when select 4 or more tooth range

- **Status**: Close

- **URL**: https://www.notion.so/2a7b6d88d2cf80ffa3efe5cf6f5bd8d1



## [1276] refund request can’t be done for approved refund. show refunded status

- **Status**: Close

- **URL**: https://www.notion.so/2a7b6d88d2cf80289341dfa0f654e24f

- **Description**: Refunded or rejected request of refund - that order can’t make refund request.

After approve or reject - admin can’t see Approve or cross reject button. 

organize, latest to oldest request.



## [1275] implant bridge no crown materials shown

- **Status**: Close

- **URL**: https://www.notion.so/2a7b6d88d2cf80beb9c6cfe58e47a762



## [1273] Handle wrong password

- **Status**: Close

- **URL**: https://www.notion.so/2a7b6d88d2cf80efa8f1c7823488a893

- **Description**: change wrong password error handle message



## [1272] Password change

- **Status**: Close

- **URL**: https://www.notion.so/2a7b6d88d2cf80698801f52b17df3646

- **Description**: password change - 404 error.
Password changed successfully but showing error



## [1271] Price change +$30

- **Status**: Close

- **URL**: https://www.notion.so/2a7b6d88d2cf80509e39e5c08d9dcbc9

- **Description**: Price change +$30



## [1270] Need alert pop-up to delete order.

- **Status**: Close

- **URL**: https://www.notion.so/2a7b6d88d2cf8015b982de1af247251c

- **Description**: Need alert pop-up to delete order.
Wordings →
Delete order
Are you sure you want to delete this order?
This action cannot be undone.
Buttons: Cancel, Delete



## [1269] Additional services not selected shown

- **Status**: Close

- **URL**: https://www.notion.so/2a7b6d88d2cf80378d9fefdbe86aa08b

- **Description**: When item is not selected in Additional services(ie. Anodize service in this case), it should not display at all.



## [1268] Design review from notification

- **Status**: Close

- **URL**: https://www.notion.so/2a7b6d88d2cf80589c1ec9b546faf3e4

- **Description**: [UI] When customer clicks “Design file added” under notification section, goes directly to “Review design” section 



## [1267] Patient name in Recent updates

- **Status**: Close

- **URL**: https://www.notion.so/2a7b6d88d2cf80118e4dfbbeb51207a5

- **Description**: Order should have order name(patient name) and order number in parentheses, eg. Design file added for the order name Michael Jordan (01-000029-JA). 



## [1266] Price Estimation design 

- **Status**: Close

- **URL**: https://www.notion.so/2a7b6d88d2cf80388b0bee195a654f41

- **Description**: Change price estimation design in order preview



## [1265] Meeting schedule notification

- **Status**: Close

- **URL**: https://www.notion.so/2a5b6d88d2cf80d7843adcb9368db286

- **Description**: no notification for schedule meeting for both side.



## [1264] Meeting schedule

- **Status**: Close

- **URL**: https://www.notion.so/2a5b6d88d2cf80779895fb3ce639d55f

- **Description**: Meeting not scheduled. Not showing in upcoming.
Both in user and admin



## [1263] chat notification

- **Status**: Close

- **URL**: https://www.notion.so/2a5b6d88d2cf80c580c5e25324376460

- **Description**: requires chat pop up in user when admin send message while user online - and chat notification of pending/unread message

test failed. only when user in home dashboard, can notify chat message. if any other page, no chat notification or any signal. and when admin send message while user on other page, and go back to home page. no alert as well.

adding alert on order details page, - chat with us
adding notification in order list, earphone icon - check image

and if user come to any another page and message unread,  keep notification



## [1262] payment notification

- **Status**: Close

- **URL**: https://www.notion.so/2a5b6d88d2cf80fcb713c88de9d0bea1

- **Description**: no transaction update from both user and admin side



## [1261] Search with invoice id

- **Status**: Close

- **URL**: https://www.notion.so/2a5b6d88d2cf800e96c1f58057b8f86c

- **Description**: user can search with invoice ID



## [1258] Missing “Shipping Label”

- **Status**: Close

- **URL**: https://www.notion.so/2a4b6d88d2cf8082ab54d13acc813249

- **Description**: For PVS Impression & Stone model



## [1257] Report not changing

- **Status**: Close

- **URL**: https://www.notion.so/2a4b6d88d2cf803d8a47c0393eedb4b6

- **Description**: Report not changing numbers



## [1256] Change FAQ text box

- **Status**: Close

- **URL**: https://www.notion.so/2a4b6d88d2cf8003a3e6d9ee42e14e2a

- **Description**: Can we change the text box to look like ones from Pricing and Credit Policy page? I want to create a bullets and numbering, but it doesn’t reflect on the page



## [1255] Updating order information issues

- **Status**: Close

- **URL**: https://www.notion.so/2a4b6d88d2cf806a8b40e8c23740b6c2

- **Description**: When backing up and changing information on the same order, it keeps the previous data and conflicts with the new data input.



## [1254] STL viewer issue

- **Status**: Close

- **URL**: https://www.notion.so/2a4b6d88d2cf80bba664e033fca8c4df

- **Description**: Issue with STL file display on viewer



## [1253] File name change

- **Status**: Close

- **URL**: https://www.notion.so/2a4b6d88d2cf8089a15edbd576ee3ed9



## [1252] Credential issue keep popping up

- **Status**: Close

- **URL**: https://www.notion.so/2a4b6d88d2cf80798d9bd4235641e87b

- **Description**: On multiple occasion, we see credential errors. Is this a test server issue?

Yes. server session issue.



## [1251] Dropdown menu not displaying

- **Status**: Close

- **URL**: https://www.notion.so/2a4b6d88d2cf80e09dc6f755f9c77807

- **Description**: Data does not display and need to refresh the page to see dropdown items

session issue.



## [1250] Additional service PMMA Crown selection

- **Status**: Close

- **URL**: https://www.notion.so/2a4b6d88d2cf8079bc2bf3b2ee3b5943

- **Description**: Unless all PMMA crown is toggled, it doesn’t activate Continue. We should be able to only partially toggle on



## [1249] Filter issue

- **Status**: Close

- **URL**: https://www.notion.so/2a4b6d88d2cf804b89eac21df73fe0b3



## [1248] Issue with next page button

- **Status**: Close

- **URL**: https://www.notion.so/2a4b6d88d2cf80dfbcadd6eb9141d140

- **Description**: Group number changes but the order details stay the same as the first page



## [1247] Continue button issue with Registration

- **Status**: Close

- **URL**: https://www.notion.so/2a4b6d88d2cf8028a8abc74849733ad2



## [1246] Reopen order overrides previously Completed order

- **Status**: Close

- **URL**: https://www.notion.so/2a4b6d88d2cf800387a5fc9e72c2ab1b

- **Description**: When “Reopen order” is clicked, new order should be created with new order # and it should not override a Completed order. Completed order should be kept as is. Consider “Reopen order” as new order with information filled.



## [1245] Dropdown doesn’t work

- **Status**: Close

- **URL**: https://www.notion.so/2a4b6d88d2cf80da81c7d80a0a580b61

- **Description**: In case of backing up order pages to edit order, the Case type 1 dropdown does not work.



## [1244] Credential error keeps popping up

- **Status**: Close

- **URL**: https://www.notion.so/2a4b6d88d2cf80778afbd7eb8db6c6cf

- **Description**: not bug, session limited by backend currently



## [1243] Stump shade displayed on Implant case

- **Status**: Close

- **URL**: https://www.notion.so/2a4b6d88d2cf80679e04e9accb9dd24c

- **Description**: Please check both Implant crown and Implant bridge



## [1242] Price doesn’t add up

- **Status**: Close

- **URL**: https://www.notion.so/2a4b6d88d2cf80bcb718da2601c918c9

- **Description**: It looks like there is duplicative charge as subtotal is $442 and final total is $462, a $20 difference

This is same with Ship PVS impression and Ship stone model.

I think there is issue with Shipping label charge calculation



## [1241] Price item need to be displayed when making payment

- **Status**: Close

- **URL**: https://www.notion.so/2a4b6d88d2cf8077ad9fed4ff9e07e22

- **Description**: Left, before payment
Right box, on Order detail Price estimate section



## [1240] Dashboard display error on Customer side

- **Status**: Close

- **URL**: https://www.notion.so/2a4b6d88d2cf8049a44ace8d5a6f18af

- **Description**: Once refreshed, fixed but displays frequently

- session limit



## [1239] Missing Ridge lap

- **Status**: Close

- **URL**: https://www.notion.so/2a4b6d88d2cf8041a50ec5a9f9a04e15

- **Description**: Missing on Implant crown & Implant bridge



## [1238] Continue button activated w/o Shipping address fill-in

- **Status**: Close

- **URL**: https://www.notion.so/2a3b6d88d2cf80af909ed0f64a299c67



## [1237] Hide Hold Order after Delivered

- **Status**: Close

- **URL**: https://www.notion.so/2a3b6d88d2cf80f9ba17f64b1688b66b



## [3005] Gift Selection Inconsistency

- **Status**: Ready for test

- **URL**: https://www.notion.so/313b6d88d2cf8038abc4cb2ce1e3a2c0



## [2983] Issue with sending the same gift twice during a video extension

- **Status**: Close

- **URL**: https://www.notion.so/30fb6d88d2cf80a58d6bce0a63d4982d



## [2243] Apple account login

- **Status**: Close

- **URL**: https://www.notion.so/2feb6d88d2cf806a8da8ee22a5f85d4a

- **Description**: Not working apple



## [2935] Server error the end of authentication

- **Status**: Close

- **URL**: https://www.notion.so/309b6d88d2cf80b09dbedc3064ef84a6



## [3003] Video call connection failure

- **Status**: Close

- **URL**: https://www.notion.so/313b6d88d2cf8024bf7fdf600b609514



## [3004] Video Connection Drop on Extension Acceptance

- **Status**: Close

- **URL**: https://www.notion.so/313b6d88d2cf8073b33bdeae5d05a351



## [2991] The purchased game currency is not being reflected

- **Status**: Close

- **URL**: https://www.notion.so/311b6d88d2cf80139d4fcc170c8b35af



## [2232] Mobile frontend details

- **Status**: Close

- **URL**: https://www.notion.so/2feb6d88d2cf80289abffb96958b180f



## [2926] Game fails to transition to Result Screen when System Tray is open

- **Status**: Close

- **URL**: https://www.notion.so/306b6d88d2cf8056b479edef7df07129



## [2989] DM notification and red dot badge not working

- **Status**: Close

- **URL**: https://www.notion.so/30fb6d88d2cf800087bae3cecbbaeb46



## [2988] Post content field cursor issue

- **Status**: Close

- **URL**: https://www.notion.so/30fb6d88d2cf80c19ccafa61c28fb878



## [2987] The input field is cutting off

- **Status**: Close

- **URL**: https://www.notion.so/30fb6d88d2cf802b915de15d6ef057df



## [2986] Restrict special characters and spaces in User Nicknames

- **Status**: Close

- **URL**: https://www.notion.so/30fb6d88d2cf80e48fe5dcb8c6f8ca5a



## [2985] Win/Loss record is not updated after matchs

- **Status**: Close

- **URL**: https://www.notion.so/30fb6d88d2cf80f484b1c822e3989421



## [2974] The captured moment shouldn't appear at all.

- **Status**: Close

- **URL**: https://www.notion.so/30db6d88d2cf8088ac39c48e4d0ca863



## [2971] Adjust swipe-to-action threshold to 40% width

- **Status**: Close

- **URL**: https://www.notion.so/30db6d88d2cf8067919dda814833ab5f



## [2931] Receiver gets no notification when a sender sends a gift

- **Status**: Close

- **URL**: https://www.notion.so/308b6d88d2cf80ba9b24dd5d88d6b13e



## [2946] Onboarding page redirection when starting app

- **Status**: Close

- **URL**: https://www.notion.so/30bb6d88d2cf8016a615c6d340ca1688



## [2345] The header navigation component differs between each tab.

- **Status**: Close

- **URL**: https://www.notion.so/2ffb6d88d2cf8050a26ed064f3b14a0d



## [2970] Connection issue in video extension

- **Status**: Close

- **URL**: https://www.notion.so/30db6d88d2cf8037860be3e89e2244e5



## [2969] Video calling and request stabilisation.

- **Status**: Close

- **URL**: https://www.notion.so/30db6d88d2cf8085b678eaa1de967938



## [2956] Follow button in match result page is not working

- **Status**: Close

- **URL**: https://www.notion.so/30cb6d88d2cf80c59eace2839f796ee2



## [2954] Repeated onboarding page

- **Status**: Close

- **URL**: https://www.notion.so/30cb6d88d2cf8051be49faaffb912d0b



## [2936] Server error on going to matching

- **Status**: Close

- **URL**: https://www.notion.so/309b6d88d2cf803fa0f5d4ee4b53c1b6



## [2234] Navigating to PasswordResetCompletion Screen after sign up

- **Status**: Close

- **URL**: https://www.notion.so/2feb6d88d2cf808ebc35dfad92dd9aee



## [2346] Rematch and Gift for video extension feature

- **Status**: Close

- **URL**: https://www.notion.so/2ffb6d88d2cf802abd94fdb9c6c9d520



## [2930] implement the UI about action icon in DM

- **Status**: Close

- **URL**: https://www.notion.so/306b6d88d2cf8004a22ac7e7395b4c38



## [2227] Duplicate users often appear in the DM list.

- **Status**: Close

- **URL**: https://www.notion.so/2feb6d88d2cf8059a911cfc691ad6aa0



## [2237] Indicating the number of new messages

- **Status**: Close

- **URL**: https://www.notion.so/2feb6d88d2cf8009bcb9dbf905236757

- **Description**: The icon indicating the number of new messages visible in the DM message list does not disappear after reading the messages.



## [2239] Messages are not showing in real-time.

- **Status**: Close

- **URL**: https://www.notion.so/2feb6d88d2cf80b1b308c08eb6d8e430

- **Description**: In the DM, messages are not showing in real-time.



## [2039] UI components are hidden by the system keyboard on mobile.

- **Status**: Close

- **URL**: https://www.notion.so/2fbb6d88d2cf8054b587c07af173920e



## [2932] No Validation message about password

- **Status**: Close

- **URL**: https://www.notion.so/308b6d88d2cf80c49c0dfd0848f7b54e



## [2230] Multiple sign up with the same email

- **Status**: Close

- **URL**: https://www.notion.so/2feb6d88d2cf8047b58ad5ed6e79c264

- **Description**: Please check a 'Duplicate Email Check' logic.



## [2925] Remove the UI in extension video

- **Status**: Close

- **URL**: https://www.notion.so/306b6d88d2cf8054a3edeabecffa6431



## [2929] The success message for extension not appear

- **Status**: Close

- **URL**: https://www.notion.so/306b6d88d2cf803c8560fda292741cfb



## [2887] Onboarding page reloads repeatedly

- **Status**: Close

- **URL**: https://www.notion.so/303b6d88d2cf80d5b527fa161af417b1



## [2909] Reset the game currency

- **Status**: Close

- **URL**: https://www.notion.so/306b6d88d2cf80628728d0130c344af6



## [2928] Anyone can access the homescreen without registing

- **Status**: Close

- **URL**: https://www.notion.so/306b6d88d2cf80b9b8bcfee44b8a90f4



## [2895] Allow admin to check the balance of each user

- **Status**: Close

- **URL**: https://www.notion.so/303b6d88d2cf80f896a3fe31cd3821cb



## [2894] Redirecting to the home screen without signing up.

- **Status**: Close

- **URL**: https://www.notion.so/303b6d88d2cf80139db3c69ca1dc366e



## [2923] UI Overlap double count down

- **Status**: Close

- **URL**: https://www.notion.so/306b6d88d2cf80028e90de3b3a3bdfbf



## [2891] Recursive Trigger countdown, blink detection

- **Status**: Close

- **URL**: https://www.notion.so/303b6d88d2cf80eb823ff5fa3ccfe2c5



## [2224] Modal

- **Status**: Close

- **URL**: https://www.notion.so/2feb6d88d2cf8045bc56e789f6acf899

- **Description**: If the other user clicks the back button without selecting anything, the request user will keep the modal window open.



## [2240] Please make the error msg

- **Status**: Close

- **URL**: https://www.notion.so/2feb6d88d2cf8066b876e32df273bb10

- **Description**: '아이디 또는 비밀번호가 일치하지 않습니다.'



## [2339] Video calling issue

- **Status**: Close

- **URL**: https://www.notion.so/2feb6d88d2cf800d81d4e63f6350ff80



## [2331] User status about ‘permanent ban and 7-days blocked’

- **Status**: Close

- **URL**: https://www.notion.so/2feb6d88d2cf800a9df2fbe9ed173432



## [2329] Account issue in the same device

- **Status**: Close

- **URL**: https://www.notion.so/2feb6d88d2cf80af8445c631f8a514bb



## [2226] redirect issue

- **Status**: Close

- **URL**: https://www.notion.so/2feb6d88d2cf803e843ac416853de52e

- **Description**: 1. when we go to the https://blink-dashboard.2.potentialai.dev it should be redirect to the https://blink-dashboard.2.potentialai.dev/login if i am not authenticated otherwise it should redirect me https://blink-dashboard.2.potentialai.dev/dashboard
2. when i am already logged in, it should redirect me to directly the dashboard.
3. also when i am already logged in, it shouldn't show the login page. if i try to go to the login page it should redirect me to the dashboard page cause i am authenticated



## [2229] Request permission

- **Status**: Close

- **URL**: https://www.notion.so/2feb6d88d2cf8045b353cfd4cf45e8de



## [2235] App password rule & The button is cut off

- **Status**: Close

- **URL**: https://www.notion.so/2feb6d88d2cf80a8b141db52aef9b28d

- **Description**: Talha, please check
I can sign up when entering a password without any symbols.

Rule:
Use 8 to 12 characters with a mix of letters, numbers, and symbols.



## [2199] camera permission error

- **Status**: Close

- **URL**: https://www.notion.so/2fcb6d88d2cf80d58479f741e94d493e

- **Description**: ‘Get a permission’ button is not working.



## [1967] Camera permission request timing issue

- **Status**: Close

- **URL**: https://www.notion.so/2fbb6d88d2cf80afa10fc1dba2236eb8

- **Description**: Currently, the camera permission is requested right after the Splash screen. Please move this to the Onboarding flow after login



## [2228] Camera screen doesn't show up when next match

- **Status**: Close

- **URL**: https://www.notion.so/2feb6d88d2cf802d81b9f31fb8ece7e1

- **Description**: Have you tested the 'Next Match' flow more than 5 times before deployment? Is it only happening for me or not.



## [2341] Shop page

- **Status**: Close

- **URL**: https://www.notion.so/2ffb6d88d2cf805b8baac5c4efb9668a



## [2241] The purchase button must display the currency to be used.

- **Status**: Close

- **URL**: https://www.notion.so/2feb6d88d2cf80b2979cfa856a6e0601



## [2225] Login again the same device

- **Status**: Close

- **URL**: https://www.notion.so/2feb6d88d2cf80508108cc7d9189d1b1

- **Description**: I still see 'This account is already logged in on another device' message



## [2038] Splash screen image is cut off

- **Status**: Close

- **URL**: https://www.notion.so/2fbb6d88d2cf80abb9d5d5c055b8bd7b



## [2242] When the text goes to the next line (line break), the keyboard covers the cursor again.

- **Status**: Close

- **URL**: https://www.notion.so/2feb6d88d2cf802784d8e33bbfc4dc6f

- **Description**: The screen should automatically scroll up as the user types more lines.



## [2244] User's country of my profile will be from their location

- **Status**: Close

- **URL**: https://www.notion.so/2feb6d88d2cf80abb6a0f10d667b2770

- **Description**: not selection



## [2246] Can't delete my post

- **Status**: Close

- **URL**: https://www.notion.so/2feb6d88d2cf8036860ff70bad0722ad



## [2247] Keyboard Overlap Issue

- **Status**: Close

- **URL**: https://www.notion.so/2feb6d88d2cf802897a8f9f4ab5fc71b

- **Description**: Buttons and input fields are obscured when the software keyboard appears.



## [2248] Modal, button, nav tab at the bottom is still covered by the phone's system navigation bar.

- **Status**: Close

- **URL**: https://www.notion.so/2feb6d88d2cf80e696d1d346aea99bd6

- **Description**: It seems like using useSafeAreaInsets from react-native-safe-area-context could fix this. Could you please check this and apply it to the modal bottom padding?



## [2249] User creation

- **Status**: Close

- **URL**: https://www.notion.so/2feb6d88d2cf80e683defc9b94c22675

- **Description**: Not match properties



## [2250] Failed to process user reports

- **Status**: Close

- **URL**: https://www.notion.so/2feb6d88d2cf808f83b4f624ebbae7a4



## [2251] Verification resend button

- **Status**: Close

- **URL**: https://www.notion.so/2feb6d88d2cf8057a587d753328b5553

- **Description**: need to click the 'Resend code' button a few times, not just once.



## [1966] Status of Account

- **Status**: Close

- **URL**: https://www.notion.so/2fbb6d88d2cf804c959cc07cffca660c

- **Description**: ‘eddy@potentialai.com’ account is activative but I can’t sign in with that



## [1570] Modal 

- **Status**: Close

- **URL**: https://www.notion.so/f11613a21b754209b7747938086fbd21

- **Description**: If the other user clicks the back button without selecting anything, the request user will keep the modal window open.



## [1582] Now work single click

- **Status**: Close

- **URL**: https://www.notion.so/8b5cab42a31247e99426bed19a31e466

- **Description**: The checkbox works with a single click, but the text ('I agree...') does not respond at all



## [1563] DM창 - online offline 관련

- **Status**: Close

- **URL**: https://www.notion.so/75ac720dddfb4d51870b1ab864cd5549

- **Description**: 상대방이 블링크에 접속해있는데도, DM창의 상대방 프로필 정보는 offline으로 떠 있음 (이 부분은 더미인걸까요!)



## [1597] My profile edit

- **Status**: Close

- **URL**: https://www.notion.so/16cac73af39a44a382721eeeb48ff4fe

- **Description**: 1. When I edit my profile, the updated ID is being displayed in the Bio section instead of the ID field.

2. The actual Bio content is not appearing at all.

3. The profile ID in the header remains unchanged and does not reflect the update.



## [1599] Navigation State Issue in Social Tab

- **Status**: Close

- **URL**: https://www.notion.so/1ea373bc30594f5a91d948932aaf54d0

- **Description**: Returning to the Social Tab displays the Settings menu instead of the main Social view.



## [1603] Inconsistent profile image across Home, Shop, and My Profile screens

- **Status**: Close

- **URL**: https://www.notion.so/840c21524f2e4d22a8bc2f8ac2ddde17

- **Description**: The user's profile image does not match across different sections of the app.



## [1595] My profile image of homeScreen does not meet my social

- **Status**: Close

- **URL**: https://www.notion.so/0d8aef4e31754359baa9cf97224a46a6

- **Description**: Even if you do not set a profile picture, the existing dummy profile will be displayed.



## [1594] Notification icon on homescreen

- **Status**: Close

- **URL**: https://www.notion.so/84ca3f1035264c9ca8b87103171bb029

- **Description**: It displays 'New Message' even when there is no new message.



## [1593] Error messages appear twice

- **Status**: Close

- **URL**: https://www.notion.so/a352e693798d4187997e0cb115f85473

- **Description**: The error message is displayed twice.



## [1575] Connection issue

- **Status**: Close

- **URL**: https://www.notion.so/013909970f96420da74750a3e2cc5638

- **Description**: The same ID can log in other device at the same time
"이 계정은 현재 다른 기기에서 이용 중이에요. 계정을 다시 한 번 확인해주세요."



## [1573] Keep showing the request permission screen.

- **Status**: Close

- **URL**: https://www.notion.so/305e404a98b34d59b1a8a9f52464d806

- **Description**: The user has already accepted the permission, but I keep showing it when they log out and then log back in.



## [1574] The new user doesn't have any currency

- **Status**: Close

- **URL**: https://www.notion.so/3afa0a07d902485780ec6a138aa87e15

- **Description**: Every new user has 2,000 Gold and 1,000 DIA in the MVP Phase



## [1571] Some UI are under the basic navigation

- **Status**: Close

- **URL**: https://www.notion.so/66abd1a7df7b482da15bf4065aa5ce4b

- **Description**: User can't click some button because that is under the navigation

Jan26: The navigation tab on the app is still under the phone's basic navigation.



## [1576] admin can't log out

- **Status**: Close

- **URL**: https://www.notion.so/a1905b5c1c9a470abe84882b356b9cb1



## [1568] The camera didn't work on Home screen

- **Status**: Close

- **URL**: https://www.notion.so/1baaa1bfbf474c3396596287f765bd59

- **Description**: User need to see their own face on the home screen



## [1562] Header Profile Layout Inconsistency (home & shop)

- **Status**: Close

- **URL**: https://www.notion.so/1e4ca5aa06b34f6789e689c405675cf9

- **Description**: Synchronize the Shop screen header's profile UI to match the Home screen exactly (CSS properties, assets, and spacing).



## [1569] endless login…

- **Status**: Close

- **URL**: https://www.notion.so/4c9a28ecd86a4dc3b86976d83104d7f3

- **Description**: I can't go to the next page on the last step of login process



## [1566] Clicking twice issue

- **Status**: Close

- **URL**: https://www.notion.so/e0894681398e4023a46553641d5340c4

- **Description**: After the page loads, 3 dots must be clicked twice to see the options.



## [2906] Make Complete Your Profile page scrollable

- **Status**: Close

- **URL**: https://www.notion.so/303b6d88d2cf814cbfc7e1acf3534e8d



## [2344] Calendly Evente link sent to email not found. Event not found. 

- **Status**: Close

- **URL**: https://www.notion.so/2ffb6d88d2cf8008a0a4c29352af0bba



## [2343] Submit button not working in level test page. 

- **Status**: Close

- **URL**: https://www.notion.so/2ffb6d88d2cf801b9e57c40b288f7b35

- **Description**: Reference image added 



## [2342] Available time selection not working 

- **Status**: Close

- **URL**: https://www.notion.so/2ffb6d88d2cf806a988ec18ecbf19082



## [1539] PUT /user/{id} returns 500 for any update

- **Status**: Close

- **URL**: https://www.notion.so/2f7b6d88d2cf81cc94d5dedb302de08d

- **Description**: The PUT /user/{id} endpoint returns 500 Internal Server Error for any update request, even minimal ones like {"name": "Test"}.

Steps to reproduce:
1. Login as admin
2. PUT /user/291 with {"name": "New Name"}
3. Returns 500 error

Expected: User profile should be updated
Actual: 500 Internal Server Error

Likely cause: Missing null check or relation handling in UserController.updateUser()



## [1380] Adding Class Information in Student Management

- **Status**: Close

- **URL**: https://www.notion.so/2bfb6d88d2cf80a08825c375d91b1bd1



## [1352] Can't update the attachment in community page .

- **Status**: Close

- **URL**: https://www.notion.so/2aeb6d88d2cf80fd8774c722084095d2

- **Description**: Failed to update the attachment . 



## [1221] Add Default Meta Tags with Dashboard Customization Option

- **Status**: Close

- **URL**: https://www.notion.so/2a2b6d88d2cf805393bfe16f6aae8601

- **Description**: Implement default meta tags for all pages and posts, and add a feature in the dashboard that allows users to customize these tags manually.



## [1138] Can’t check carousel images in community detail page

- **Status**: Close

- **URL**: https://www.notion.so/29db6d88d2cf8098889fc5836d4c4d71



## [1137] We don’t need time for start_date

- **Status**: Close

- **URL**: https://www.notion.so/29db6d88d2cf8094bc26def2b48b58de



## [1119] Sorting with Text Book is not working properly

- **Status**: Close

- **URL**: https://www.notion.so/29cb6d88d2cf80e78f29c6f66ffb2dd9



## [1118] Please remove ‘Post Details’ in Community detail page

- **Status**: Close

- **URL**: https://www.notion.so/29cb6d88d2cf8069b030f06795621985



## [1117] Line Break is not working

- **Status**: Close

- **URL**: https://www.notion.so/29cb6d88d2cf80aeaad6f15bab30519b



## [1116] - Birth of Year, Phone number must be optional

- **Status**: Close

- **URL**: https://www.notion.so/29cb6d88d2cf80ec8a74c6d75e511f21



## [1044] Changing Hero Section

- **Status**: Close

- **URL**: https://www.notion.so/295b6d88d2cf80e39719f3e3aa4a2be8



## [1029] We can remove Trial Class in Pricing Page

- **Status**: Close

- **URL**: https://www.notion.so/293b6d88d2cf804ea5bdee3a69bec5cf



## [1028] Phone Number Field should be splitted to country code & phone number

- **Status**: Close

- **URL**: https://www.notion.so/293b6d88d2cf80259c88dd203d347e47



## [1025] I can’t find Where I can check Enroll information

- **Status**: Close

- **URL**: https://www.notion.so/293b6d88d2cf8041955cf78dab9002e4



## [1024] I can’t find Where I can check Contact Us information

- **Status**: Close

- **URL**: https://www.notion.so/293b6d88d2cf80b5aa7be35bd7079843



## [1021] Bots are submitting application too many times

- **Status**: Close

- **URL**: https://www.notion.so/292b6d88d2cf80ff8782cdd8c0d42382



## [1020] Training Center → Language Center

- **Status**: Close

- **URL**: https://www.notion.so/292b6d88d2cf8052b576dcbe4f6565c2



## [1012] Thumbnail image not working on Community page

- **Status**: Close

- **URL**: https://www.notion.so/9ec68147ee0d4e369cd38d1545b4ed8e

- **Description**: In the Ktalk Live project, the thumbnail image is not working on the Community page.



## [1011] Community Post is not clickable, when not loged in. (it should be shown to not logged in user)

- **Status**: Close

- **URL**: https://www.notion.so/28fb6d88d2cf801a8e43f13897b8ee3b



## [978] UI Issue

- **Status**: Close

- **URL**: https://www.notion.so/261b6d88d2cf80b181d3dd7048d8353c



## [977] Can’t use Coupon here

- **Status**: Close

- **URL**: https://www.notion.so/261b6d88d2cf803da12cf933d38eae60



## [969] When I change status of applicant, It redirects to 1st Page, but pagination is not correct

- **Status**: Close

- **URL**: https://www.notion.so/255b6d88d2cf80fd9900e40a614e9b01



## [968] Change SMTP Email to team@ktalk.llive

- **Status**: Close

- **URL**: https://www.notion.so/255b6d88d2cf80d3a015e500c1af1dae



## [960] Please add Country In Residence

- **Status**: Close

- **URL**: https://www.notion.so/252b6d88d2cf80ceb829cb116522d78e



## [959] Please use proper english

- **Status**: Close

- **URL**: https://www.notion.so/252b6d88d2cf80628e0fee43f1853e1f



## [958] Please show only current textbook

- **Status**: Close

- **URL**: https://www.notion.so/252b6d88d2cf8072a760cd045d4f8d4d



## [957] Error message is showing

- **Status**: Close

- **URL**: https://www.notion.so/252b6d88d2cf8088b198dc63476c14af



## [956] Add Edit Student Icon

- **Status**: Close

- **URL**: https://www.notion.so/252b6d88d2cf80c08e3de9caf9ae255e



## [954] Make Student Name Clickable

- **Status**: Close

- **URL**: https://www.notion.so/252b6d88d2cf80df9a5be2cbe4098e47



## [939] Teacher can’t update Text Book

- **Status**: Close

- **URL**: https://www.notion.so/24fb6d88d2cf806287ece707d22bfc86



## [936] Ending Date is not Shoiwng

- **Status**: Close

- **URL**: https://www.notion.so/24fb6d88d2cf8094bbf8f10c060422df



## [935] Please add Country of Residence

- **Status**: Close

- **URL**: https://www.notion.so/24fb6d88d2cf80e280e3f5fade4b3e1c



## [894] Payment Page, Please change 1 month → 4 weeks

- **Status**: Close

- **URL**: https://www.notion.so/247b6d88d2cf807c9d4ff8e442ca379a



## [893] Payment Page, Please remove Trial / Free class

- **Status**: Close

- **URL**: https://www.notion.so/247b6d88d2cf80feb1a5c5cf795c0dd0



## [892] Please remove Class in Applicant Details page

- **Status**: Close

- **URL**: https://www.notion.so/247b6d88d2cf8032af55c5cf917e5f1b



## [891] Information is not showing

- **Status**: Close

- **URL**: https://www.notion.so/247b6d88d2cf805781e8f6bbd2624ed7



## [890] Tutor not found

- **Status**: Close

- **URL**: https://www.notion.so/247b6d88d2cf80629bd1ff7551efa507



## [889] Today is not supposed to be class date

- **Status**: Close

- **URL**: https://www.notion.so/247b6d88d2cf80188d6ad78b7f58986f



## [888] Student is not being assigned

- **Status**: Close

- **URL**: https://www.notion.so/247b6d88d2cf803fadd8cf8287a3fdc8



## [887] Day / Time / Class Time is removed

- **Status**: Close

- **URL**: https://www.notion.so/247b6d88d2cf8071b5c3ed3a505e0895



## [864] Textbook edit functionality not work

- **Status**: Close

- **URL**: https://www.notion.so/246b6d88d2cf809b8107e48a8f409fc7



## [861] Edit Class Page:- Students is not showing

- **Status**: Close

- **URL**: https://www.notion.so/245b6d88d2cf8031b3e0cea611100acf



## [778] Change the calendar logos

- **Status**: Close

- **URL**: https://www.notion.so/237b6d88d2cf80aea400d1c42720fd08



## [757] Please add status of class
• active
• inactive

- **Status**: Close

- **URL**: https://www.notion.so/230b6d88d2cf80d49159e241ad935bd0



## [755] When the user is giving his available time, it is showing incorrectly.

- **Status**: Close

- **URL**: https://www.notion.so/22cb6d88d2cf80caaaacfd8dec3a2082



## [754] When a user makes a payment, it is not being added to the income in the admin panel.

- **Status**: Close

- **URL**: https://www.notion.so/22bb6d88d2cf8056ac71dee7d34af9c1



## [752] The word email is written twice here during email verification.

- **Status**: Close

- **URL**: https://www.notion.so/22bb6d88d2cf80329576d049c055831a



## [751] I think when a user registers, the reset button of this website has become too big in this place which is not user friendly. Users are clicking this button repeatedly by mistake, so we need to make it user friendly, but I am giving an example.

- **Status**: Close

- **URL**: https://www.notion.so/22bb6d88d2cf8081ab59cb75d41c23cc



## [750] Rearrange the columns here.

- **Status**: Close

- **URL**: https://www.notion.so/22bb6d88d2cf800682dbd41fce7cc33d



## [749] This will be a class form, not a community.

- **Status**: Close

- **URL**: https://www.notion.so/22bb6d88d2cf80bda2f9c9868869bdf7



## [748] Recent community posts are not appearing in the admin dashboard.

- **Status**: Close

- **URL**: https://www.notion.so/22bb6d88d2cf80a38bf4f986584c8074



## [747] Make a change from Add a Allowance to Paid Allowance in the >admin panel only.<

- **Status**: Close

- **URL**: https://www.notion.so/22bb6d88d2cf80cd9b86c3336878c667



## [746] Use the delete icon that is used throughout our entire application.

- **Status**: Close

- **URL**: https://www.notion.so/22bb6d88d2cf803ba5b3cf4108484d56



## [745] Depending on the class type, the units should be given in the dropdown, such as when creating a class, after selecting the class type, the units or class times appear in the dropdown.

- **Status**: Close

- **URL**: https://www.notion.so/22bb6d88d2cf807e8f4ef4eddaa4fdc7



## [] Notification issues

- **Project**: Roadmap

- **URL**: https://www.notion.so/2cbb6d88d2cf80b49b29d39735831222

- **Description**: Signup shows old notification data



## [] notification

- **Project**: K Talk

- **URL**: https://www.notion.so/1b0b6d88d2cf8000800ad07237129bbc



## [] Push notifications should be delivered regardless of phone state

- **Project**: Roadmap

- **URL**: https://www.notion.so/2ccb6d88d2cf80019b04fe1f72ebf477

- **Description**: Push notifications not working when screen off or app closed/background



## [] Notification is not clickable

- **Project**: K Talk

- **URL**: https://www.notion.so/1eab6d88d2cf804ab48cdec313058633



## [] Notification titles are not providing accurate information

- **Project**: Near Work

- **URL**: https://www.notion.so/238b6d88d2cf801ca8d6ca85c0f10ff5

- **Description**: Notification titles inaccurate. Not receiving notifications.



## [] Every time user receives notification, a recommended job is created

- **Project**: Near Work

- **URL**: https://www.notion.so/238b6d88d2cf80e6b31eed255db60a41

- **Description**: Notification & Recommended job not working properly



## [] Notifications are not sent

- **Project**: K Talk

- **URL**: https://www.notion.so/1dfb6d88d2cf8094a69ddb16afe2590a



## [] Notification icon placement

- **Project**: K Talk

- **URL**: https://www.notion.so/1b1b6d88d2cf809e8fe8ed915cf3e6ba



## [] No follow-up action/notification with Refund request

- **Project**: Ivory

- **URL**: https://www.notion.so/2a3b6d88d2cf80cfbb38ec49480ee4e4



## [1780] notification number need the actual numbers

- **Project**: Coaching Record

- **Status**: Close

- **URL**: https://www.notion.so/5dc9d000147c4ee285a1404e5aaf8dbc



## [] Notification shows two times

- **Project**: Ivory

- **URL**: https://www.notion.so/2a1b6d88d2cf80938b28f5a07013f63c



## [] Notification Page Not working

- **Project**: Ivory

- **URL**: https://www.notion.so/2a1b6d88d2cf805cbe05f07805b30e1c



## [] Notification number cut and static

- **Project**: Coaching Record

- **URL**: https://www.notion.so/58863d88138a4ecb812365d9b9bff84d



## [] Notification number cut and static (2)

- **Project**: Coaching Record

- **URL**: https://www.notion.so/7dc971e84b7947dd97391645078a8687



## [] Notification message is not Shown properly

- **Project**: Coaching Record

- **URL**: https://www.notion.so/2fbb6d88d2cf81a89ef0d235d9538533



## [] No new notification but showing red dot

- **Project**: Ivory

- **URL**: https://www.notion.so/2a3b6d88d2cf807897b9d19fe48f33f7



## [] Mark all as read in Notification Is not working

- **Project**: Coaching Record

- **URL**: https://www.notion.so/2fbb6d88d2cf8172169f9872768cefb



## [] There is no notification but still showing red dot

- **Project**: Ivory

- **URL**: https://www.notion.so/29cb6d88d2cf80aa8231dc4603674ddf



## [1743] Chat notification count number not clearing

- **Project**: Coaching Record

- **Status**: Close

- **URL**: https://www.notion.so/1edd0154f61b4acc98b919e41cd3ae24



## [] Upload New File Issues

- **Project**: Elite4Print

- **URL**: https://www.notion.so/2feb6d88d2cf808d99aac775b30314fa



## [] Upload File Requirement Issue

- **Project**: Elite4Print

- **URL**: https://www.notion.so/2feb6d88d2cf807ebb6cdc102e401a55



## [180] uploading photos

- **Project**: Entermong

- **Status**: Close

- **URL**: https://www.notion.so/1bbb6d88d2cf80689b36e0990853f27c

- **Description**: Photos uploaded without app requesting permission to read gallery



## [] Wrong UI and file upload on Approved design STL

- **Project**: Ivory

- **URL**: https://www.notion.so/2a1b6d88d2cf80f684c9f8460167fa8a



## [] fix image viewer

- **Project**: Ivory

- **URL**: https://www.notion.so/298b6d88d2cf808b90a6f1d5d9712d5d



## [] Profile image didn't change

- **Project**: K Talk

- **URL**: https://www.notion.so/1dfb6d88d2cf808f842be8b661abc514



## [1462] notice image not showing in the app

- **Project**: Roadmap

- **Status**: Close

- **URL**: https://www.notion.so/2cdb6d88d2cf801c9ca4f6d5e70b8840



## [] Changing profile picture on iphone/Android by camera crashes app

- **Project**: Coaching Record

- **URL**: https://www.notion.so/a342324b53654fec86b7d74965ed8784



## [] Changing profile picture on iphone/Android by camera crashes app (2)

- **Project**: Coaching Record

- **URL**: https://www.notion.so/f63ed82bc8af40afaedd715cba4b43af



## [] Need a pagination system

- **Project**: Near Work

- **URL**: https://www.notion.so/239b6d88d2cf80ea2f3c948604a0943

- **Description**: user and admin panel pagination not working



## [] Pagination and search state reset after navigating back

- **Project**: Ivory

- **URL**: https://www.notion.so/2bfb6d88d2cf80999e33ebee7c384424

- **Description**: Back button resets page number and search results



## [] need Sorting by created at

- **Project**: Onui CRM

- **URL**: https://www.notion.so/229b6d88d2cf805c9709dd256ef59641

- **Description**: Latest items need to go to top



## [] website blog page don't have pagination

- **Project**: Thrll

- **URL**: https://www.notion.so/2a8b6d88d2cf802ea012f5786b97d034



## [] Order Status Update Not Reflected Instantly

- **Project**: Ivory

- **URL**: https://www.notion.so/23ab6d88d2cf80908906c34b7210e1c4



## [] All Filter in Order List Displays Only Pending Status

- **Project**: Ivory

- **URL**: https://www.notion.so/23ab6d88d2cf80debe6bc6cbd38eb0c8



## [] Information missing in Order edit / order details

- **Project**: Ivory

- **URL**: https://www.notion.so/29cb6d88d2cf8085b186d6a01a88158c



## [] New user should be display top of the list

- **Project**: Onui CRM

- **URL**: https://www.notion.so/216b6d88d2cf8010ab6ccf2dc3a342a8

- **Description**: New user not displayed at top of list



## [] Cancel order Error message and missing data

- **Project**: Ivory

- **URL**: https://www.notion.so/2a1b6d88d2cf8080a306dc8c3bb0a746



## [] Edit order selection/summary discrepancy

- **Project**: Ivory

- **URL**: https://www.notion.so/29db6d88d2cf80ed8bd6e38b19b4628b



## [] Searce bar is not working properly for Order List Page

- **Project**: Ivory

- **URL**: https://www.notion.so/23ab6d88d2cf802a8d90ca9b08d24922



## [533] Responsive Design, Hamburger menu icon is weird

- **Project**: Bright Pixel

- **Status**: Close

- **URL**: https://www.notion.so/210b6d88d2cf80ff8392e44e99cc8d37

- **Description**: Hamburger Menu Icon Search Icon Padding issues



## [1140] Wrong digital analog logic

- **Project**: Ivory

- **Status**: Close

- **URL**: https://www.notion.so/29db6d88d2cf8028b49fe58f1a538192

- **Description**: Digital analog should not be displayed but included based on implant position



## [1402] Parking lot UI issue still same

- **Project**: Roadmap

- **Status**: Close

- **URL**: https://www.notion.so/2c4b6d88d2cf80c9a993fb2d01dbc52c

- **Description**: Alignment and padding issues



## [328] parent management main list sorting needs to be descending order

- **Project**: Onui CRM

- **Status**: Close

- **URL**: https://www.notion.so/1e9b6d88d2cf803ab670cb64e9829442



## [] dashboard payment amount different with payment history

- **Project**: Roadmap

- **URL**: https://www.notion.so/2c7b6d88d2cf802696fddce80f26dade

- **Description**: Dashboard showing wrong payment amount (3 won vs 400,000+)



## [] Payment

- **Project**: K Talk

- **URL**: https://www.notion.so/1b0b6d88d2cf80f3817cff3b8cc2f9ee



## [] I didn't payment complete, but payment history has data

- **Project**: Roadmap

- **URL**: https://www.notion.so/2cbb6d88d2cf80afa25ec22313190350

- **Description**: Just registered card, not proceeded payment but history shows data



## [] App payment information table issue

- **Project**: Roadmap

- **URL**: https://www.notion.so/2c9b6d88d2cf808c81c2c1f4b5f31bde

- **Description**: Fields for free usage time and hourly price issues



## [] Invoice button display

- **Project**: Ivory

- **URL**: https://www.notion.so/29cb6d88d2cf8081aff9e70852752ea5



## [] Some payment redirecting to payment page again from stripe

- **Project**: Thrll

- **URL**: https://www.notion.so/298b6d88d2cf803191f2c6bb36c9d44b



## [] payment history still not showing

- **Project**: Roadmap

- **URL**: https://www.notion.so/2c4b6d88d2cf80168a74e360fce6e4ef



## [] Even the payment is cancelled it is showing cancel payment and current payment

- **Project**: Ivory

- **URL**: https://www.notion.so/2a0b6d88d2cf802ab733ca20d096477a



## [] pontic crown not counted in the price

- **Project**: Ivory

- **URL**: https://www.notion.so/29eb6d88d2cf806f827ac783350b129a



## [] payment cancellation feature

- **Project**: K Talk

- **URL**: https://www.notion.so/1bfb6d88d2cf809691eceed93993c171



## [] already paid this service issue

- **Project**: Roadmap

- **URL**: https://www.notion.so/2c9b6d88d2cf801f9cb5e36ed156551d

- **Description**: Paid vehicle still appears in search results



## [1674] Payment success is showing raw message

- **Project**: Coaching Record

- **Status**: Close

- **URL**: https://www.notion.so/de6fc71f881e433da1b7324736ede380



## [673] Not getting success message after payment

- **Project**: Bright Pixel

- **Status**: Close

- **URL**: https://www.notion.so/222b6d88d2cf803f8840c964535d17dc

- **Description**: No success message or redirect after payment. No price data on page.



## [2094] Coupon Code / Promotion — Unable to check out

- **Project**: Elite4Print

- **Status**: Close

- **URL**: https://www.notion.so/2fbb6d88d2cf81759e50f7bf68cbee8d

- **Description**: Can not check out with coupon code. Cart shows 3 decimal places. Proceed to Checkout results in error.



## [2286] Local Delivery Fees

- **Project**: Elite4Print

- **Status**: Close

- **URL**: https://www.notion.so/2feb6d88d2cf80648315e4ef84335754

- **Description**: Local delivery fees not shown in invoice



## [2309] Add Calculated Shipping Weight to Work Order

- **Project**: Elite4Print

- **Status**: Close

- **URL**: https://www.notion.so/2feb6d88d2cf8068af14cc7a25b796d8

- **Description**: Need calculated vs actual shipping weights/rates comparison



## [] Missing Shipping price in content management / price estimate

- **Project**: Ivory

- **URL**: https://www.notion.so/29fb6d88d2cf8067bdb5ca63613c1845



## [1478] Need to check Calllog management filter

- **Project**: Onui CRM

- **Status**: Ready for test

- **URL**: https://www.notion.so/2e7b6d88d2cf807d835cfb459f355f99

- **Description**: Filter is not apply



## [1401] Map marker issues

- **Project**: Roadmap

- **Status**: Close

- **URL**: https://www.notion.so/2c4b6d88d2cf80da8d70de310ba09107

- **Description**: Should be list view



## [186] notice creation

- **Project**: Entermong

- **Status**: Close

- **URL**: https://www.notion.so/1bcb6d88d2cf8006848fe3d96e478b97

- **Description**: Notice creation success message shown but missing in notice list



## [869] lead revival playbook activity log

- **Project**: Onui CRM

- **Status**: Close

- **URL**: https://www.notion.so/247b6d88d2cf80ccb2c0f75f48014bad

- **Description**: Activity log should be revival, not just update



## [508] task reminder message is not working

- **Project**: Onui CRM

- **Status**: Close

- **URL**: https://www.notion.so/210b6d88d2cf8023850ce1f69080f24c



## [325] Need to change text format

- **Project**: K Talk

- **Status**: Close

- **URL**: https://www.notion.so/1dfb6d88d2cf80feb13ee609f48a2551



## [145] Notice page

- **Project**: K Talk

- **Status**: Close

- **URL**: https://www.notion.so/1b1b6d88d2cf80c4af48ebbf03f94e69

- **Description**: Need to differentiate read/unread notices, change date format



## [1452] Reward request cannot possible to 1,000 amount

- **Project**: Roadmap

- **Status**: Close

- **URL**: https://www.notion.so/2cbb6d88d2cf8032adb7e99d3b30e7ca

- **Description**: Amount should be 5,000/10,000/15,000



## [1424] lead create api has some problem

- **Project**: Onui CRM

- **Status**: Close

- **URL**: https://www.notion.so/2c9b6d88d2cf803d89b5e41a4f2586e1

- **Description**: Lead not created but returns success message



## [244] problem while creating new lead

- **Project**: Onui CRM

- **Status**: Close

- **URL**: https://www.notion.so/1d7b6d88d2cf8072a26cecb682316bfe

- **Description**: Problem happening after the fix



## [1465] parking lot detail page design detail

- **Project**: Roadmap

- **Status**: Close

- **URL**: https://www.notion.so/2d2b6d88d2cf802d9da3cc64e86e0cf6

- **Description**: icon align, text size issues



## [1489] call log management page

- **Project**: Onui CRM

- **Status**: Close

- **URL**: https://www.notion.so/2e8b6d88d2cf8033a296fd72b19e8c30

- **Description**: Diagnosis filter auto reset, duplicate date filter, column size mismatch



## [561] auth-payment

- **Project**: K Talk

- **Status**: Close

- **URL**: https://www.notion.so/21bb6d88d2cf807e824de69e23665b2c

- **Description**: Pricing section layout needs reorganization



## [441] UI Issue in Class Detail Page

- **Project**: K Talk

- **Status**: Close

- **URL**: https://www.notion.so/201b6d88d2cf80d081c2e481fdb86cf6

- **Description**: Student List weird when no profile image set up



## [690] There is no real images of Jeju Campus

- **Project**: K Talk

- **Status**: Close

- **URL**: https://www.notion.so/223b6d88d2cf8071b2e5cbef5f7bdd94

- **Description**: Missing real images for Jeju Campus



## [579] All of contact us button should be - enroll now

- **Project**: K Talk

- **Status**: Close

- **URL**: https://www.notion.so/21bb6d88d2cf8037b755d38c941292a1

- **Description**: All buttons should be 'Enroll Now' and redirect to registration page



## [341] Change Title of Class in My Schedule

- **Project**: K Talk

- **Status**: Close

- **URL**: https://www.notion.so/1eab6d88d2cf806fa484fdc2aef162aa

- **Description**: Class title format change needed



## [1490] lead management page

- **Project**: Onui CRM

- **Status**: Close

- **URL**: https://www.notion.so/2e8b6d88d2cf80378406d9fb2f925d8c

- **Description**: diagnosis auto reset



## [835] Schedule management Team/cell

- **Project**: Onui CRM

- **Status**: Close

- **URL**: https://www.notion.so/23ab6d88d2cf80bea7c6e2bcc8074388

- **Description**: Showing 1depth/2depth instead of 3depth/4depth



## [1289] Can't create the Task

- **Project**: Onui CRM

- **Status**: Close

- **URL**: https://www.notion.so/2a8b6d88d2cf804793c9dcfea5f81181

- **Description**: Error: lost my lead funnel



## [1290] Zoom limitation

- **Project**: Roadmap

- **Status**: Close

- **URL**: https://www.notion.so/2a8b6d88d2cf80b784bfec47ebd09354

- **Description**: Need to extend zoom



## [844] Send Tally form message template

- **Project**: Onui CRM

- **Status**: Close

- **URL**: https://www.notion.so/23eb6d88d2cf80b8ab2cdbc5bd3fc963

- **Description**: Need to delete red box



## [] Some UI differences in Signup/Login pages

- **Project**: Coaching Record

- **URL**: https://www.notion.so/9806b7d2f4474ae1a21bbc8af332de7b



## [] Social signup one screen missing

- **Project**: Coaching Record

- **URL**: https://www.notion.so/7a8a62bdc1b04289a197105a512b0edd



## [] Social signup one screen missing (2)

- **Project**: Coaching Record

- **URL**: https://www.notion.so/e0978ff263ed46dca2b8ef8aab044254



## [] stuck at signup

- **Project**: K Talk

- **URL**: https://www.notion.so/1b1b6d88d2cf80628ab9e5719bfd6176



## [] Login button not responding

- **Project**: Coaching Record

- **URL**: https://www.notion.so/8afe51b8c5e84a429b917b236e7759f4



## [] Showing signup instead of Sign in or Log in

- **Project**: Thrll

- **URL**: https://www.notion.so/28db6d88d2cf802a9848dde8464b2b33



## [] Login issue in client Server-site

- **Project**: K Talk

- **URL**: https://www.notion.so/1dfb6d88d2cf80a99954f68025d11e07



## [517] Sign Up is not working

- **Project**: K Talk

- **Status**: Close

- **URL**: https://www.notion.so/210b6d88d2cf8061bae4d83e7c3ac1be



## [895] re-call playbook: tomorrow same time option

- **Project**: Onui CRM

- **Status**: Close

- **URL**: https://www.notion.so/248b6d88d2cf8084a829c559bad7d312

- **Description**: Time should be same



## [689] Student Testimonial video is static

- **Project**: K Talk

- **Status**: Close

- **URL**: https://www.notion.so/223b6d88d2cf80d0965dde24abe8d921

- **Description**: Need to change with new video



## [] Search with teacher is not working

- **Project**: K Talk

- **URL**: https://www.notion.so/21eb6d88d2cf80778c52f6f0f6fd9795

- **Description**: Filter is not working



## [] Class schedules delete option is not working

- **Project**: K Talk

- **URL**: https://www.notion.so/1dfb6d88d2cf80879552cd4040f47334



## [] Class forum is not working

- **Project**: K Talk

- **URL**: https://www.notion.so/1eab6d88d2cf80dc9fbcda2bd748ca1e

- **Description**: Created class forum but not showing



## [] Lost Trigger is not working

- **Project**: Onui CRM

- **URL**: https://www.notion.so/24cb6d88d2cf80c5a288ccd7e1bc1f65

- **Description**: Submitted re-call playbook 3 times but status not changing to C.lost



## [] Admin Logout not working

- **Project**: Ivory

- **URL**: https://www.notion.so/294b6d88d2cf80b5afd2fb79f85b1b15



## [] preview is not working

- **Project**: Roadmap

- **URL**: https://www.notion.so/2c7b6d88d2cf8089b83adccea5227a94



## [] Permission management: Call log edit is not working

- **Project**: Onui CRM

- **URL**: https://www.notion.so/23eb6d88d2cf8036bc0afdc83e08f27f

- **Description**: Call log permission checked but not working



## [] lead list page Diagnosis submit filter is not working

- **Project**: Onui CRM

- **URL**: https://www.notion.so/2e7b6d88d2cf80dbb3dfe403a78b67e8

- **Description**: Filter is not working



## [] Past 24 hour not working

- **Project**: Thrll

- **URL**: https://www.notion.so/298b6d88d2cf809099accfcf10388424



## [] Cancel subscription is not working

- **Project**: Ivory

- **URL**: https://www.notion.so/29cb6d88d2cf80768326de17c3cd1823



## [] Translation is not working properly

- **Project**: K Talk

- **URL**: https://www.notion.so/295b6d88d2cf80bea102d21899c3028c



## [] onboarding is not working properly

- **Project**: Ivory

- **URL**: https://www.notion.so/29ab6d88d2cf8079960df937adfb566f



## [] Activity log, not working properly

- **Project**: Onui CRM

- **URL**: https://www.notion.so/229b6d88d2cf80ea8dafd315a3363e5a



## [] SQL Not Completed → C.LOST → option missing

- **Project**: Onui CRM

- **URL**: https://www.notion.so/2e7b6d88d2cf804bbdcfe3b52a67d13e



## [] Upload approved designed STL not working

- **Project**: Ivory

- **URL**: https://www.notion.so/2a3b6d88d2cf8027b61fc84552fa3c77



## [] Clicking task briefly shows incorrect info

- **Project**: Onui CRM

- **URL**: https://www.notion.so/209b6d88d2cf808dbb69f5b473c3402b

- **Description**: Incorrect/old task info briefly appears before correct data loads



## [] Myprofile edit password: Validation is not working

- **Project**: Onui CRM

- **URL**: https://www.notion.so/225b6d88d2cf80d7919fc0bddd3d2675



## [] Input fields are not working

- **Project**: Trustix

- **URL**: https://www.notion.so/2f5b6d88d2cf800e9dd3c3dda81e972d



## [2944] Inconsistent Navbar Across Dashboard Pages

- **Project**: Design Flow

- **Status**: Close

- **URL**: https://www.notion.so/30bb6d88d2cf810d8db2f8795dc753b6

- **Description**: Side navbar not consistent across all dashboard pages



## [404] funnel and status

- **Project**: Onui CRM

- **Status**: Close

- **URL**: https://www.notion.so/1f9b6d88d2cf80eb877cdc46ff005fce

- **Description**: Mismatches between parent and child values of funnels and statuses



## [] Chat Page

- **Project**: K Talk

- **URL**: https://www.notion.so/26fb6d88d2cf80929c08cc3b540da1df



## [] chat text running over

- **Project**: Ivory

- **URL**: https://www.notion.so/2a1b6d88d2cf8092b35be4d3bd0b22b6



## [] Chat restricted. Chat person name incorrect.

- **Project**: Coaching Record

- **URL**: https://www.notion.so/06b45a7d423f4afe9b6a6f92fa0bcf04



## [] Message is wrong

- **Project**: Ivory

- **URL**: https://www.notion.so/2aab6d88d2cf80258c2afd6150395eb2



## [684] Zoom class API needs to be fixed

- **Project**: K Talk

- **Status**: Close

- **URL**: https://www.notion.so/223b6d88d2cf80faba10c0110f21e7e8



## [564] Please check all of copywriting using ChatGPT

- **Project**: K Talk

- **Status**: Close

- **URL**: https://www.notion.so/21bb6d88d2cf80beba4ee8eed8a55e1e

- **Description**: Copywriting corrections needed



## [525] Admin can't upload live link

- **Project**: Bright Pixel

- **Status**: Close

- **URL**: https://www.notion.so/210b6d88d2cf80b2aa8cd0e270a16573

- **Description**: Live link in e-commerce but not in dashboard



## [563] Fix HomePage

- **Project**: K Talk

- **Status**: Close

- **URL**: https://www.notion.so/21bb6d88d2cf806d9758e253395ef106



## [1701] notification number need actual numbers (2)

- **Project**: Coaching Record

- **Status**: Close

- **URL**: https://www.notion.so/18bb054f375d40b2a7b9c6bbf6d5ebfb



## [1448] Operation hour: Other information field

- **Project**: Roadmap

- **Status**: Close

- **URL**: https://www.notion.so/2cbb6d88d2cf80f5b3ece5810ec35782

- **Description**: Too many parking lots have dummy data in other information field



## [1313] Non-Contact system is not working

- **Project**: Onui CRM

- **Status**: Close

- **URL**: https://www.notion.so/2a9b6d88d2cf80da8185ef65818cf478

- **Description**: Non-contact logging not working for leads with no call logs



## [1173] need help? part missing at bottom right side

- **Project**: Ivory

- **Status**: Close

- **URL**: https://www.notion.so/2a0b6d88d2cf80fab989e751a25ef099



## [] MQL - status

- **Project**: Onui CRM

- **URL**: https://www.notion.so/218b6d88d2cf80678d7aeb064759a468

- **Description**: MQL stats only have Diagnosis not submitted. All status should be visible.



## [] Task Status and Priority

- **Project**: Ivory

- **URL**: https://www.notion.so/2a9b6d88d2cf8051a9a8d86863a62540

- **Description**: Created task without Status and Priority



## [] Parking lot detail page, realtime section

- **Project**: Roadmap

- **URL**: https://www.notion.so/2cab6d88d2cf80cd9936f044127ca609



## [] If existed user tries to signup

- **Project**: Ivory

- **URL**: https://www.notion.so/2a4b6d88d2cf802e9502e8a7b6b2f9b4

- **Description**: Existing user can go through signup, takes name/phone then redirects to homepage



## [1094] Office Phone number Error issue

- **Project**: Thrll

- **Status**: Close

- **URL**: https://www.notion.so/29cb6d88d2cf809aa4dcd562aa3e6979

- **Description**: Need to show how to input phone numbers or automatically converts to corrected input when customer inputs 10 digits.



## [371] playbook error

- **Project**: Playbook

- **Status**: Close

- **URL**: https://www.notion.so/1f4b6d88d2cf8071bc9dfdbed7336df9



## [437] UI Issue in dashboard

- **Project**: Student Dashboard

- **Status**: Close

- **URL**: https://www.notion.so/201b6d88d2cf80a69578f42332bba274

- **Description**: UI Issue - Good Morning: What if it's not morning, Class Number is showing two times, Class Type N/A



## [986] Language Error

- **Project**: Thrll

- **Status**: Close

- **URL**: https://www.notion.so/26cb6d88d2cf8034bb4bdd60299bff8d

- **Description**: After selecting `English` giving error in `Korean` language



## [1486] Filter issue

- **Project**: Playbook

- **Status**: Close

- **URL**: https://www.notion.so/2e8b6d88d2cf8074aa01d91ecf3b7217

- **Description**: When I set and reopen the filter modal: Diagnosis reset, Call result reset, Data disappear



## [1485] filter issue

- **Project**: Playbook

- **Status**: Close

- **URL**: https://www.notion.so/2e8b6d88d2cf8012ad1efdef367983d5

- **Description**: When I check the diagnosis filter and reopen the filter modal, it resets automatically. When I set date filter and reopen, date is disappear



## [1130] UI Issue

- **Project**: Thrll

- **Status**: Close

- **URL**: https://www.notion.so/29cb6d88d2cf80088656cc09d48d41f2

- **Description**: Padding needs to be correct



## [1195] UI Issue

- **Project**: Thrll

- **Status**: Close

- **URL**: https://www.notion.so/2a1b6d88d2cf80b4b9facd02ad610b2f

- **Description**: right side has much space then the left side.



## [987] Redirecting Issue

- **Project**: Thrll

- **Status**: Close

- **URL**: https://www.notion.so/26cb6d88d2cf80759239fcc1b59c9643

- **Description**: I signed up with one email and ditched the `Adventure DNA test`, after that I again signup with another email(not same email) it is redirecting me to the DNA test page where I left off.



## [841] Permission Management: Check the Create lead

- **Project**: Playbook

- **Status**: Close

- **URL**: https://www.notion.so/23eb6d88d2cf80788d52e8a13c479bb4

- **Description**: can't create the lead



## [1309] Error message is wrong

- **Project**: Thrll

- **Status**: Close

- **URL**: https://www.notion.so/2a8b6d88d2cf8030a4aee92ff3bc2af8

- **Description**: It should show `You cannot select more than 4 Featured profile`



## [238] can't change status stop to in progress

- **Project**: Student Dashboard

- **Status**: Close

- **URL**: https://www.notion.so/1d6b6d88d2cf80a6ae1cc8542f4b2f59

- **Description**: it's should possible



## [624] Lead child grade follows the customer child grade

- **Project**: Playbook

- **Status**: Close

- **URL**: https://www.notion.so/21fb6d88d2cf80da83d3cb18768513ca

- **Description**: The child grade of the lead is automatically updated to match the customer's child grade, so when the customer's child grade changes, the lead's child grade also changes.



## [827] change the word, and add the feature

- **Project**: Playbook

- **Status**: Close

- **URL**: https://www.notion.so/23ab6d88d2cf80519700cd6fdb58a940

- **Description**: change the word: ID/PW/슬랙ID → 이름/ID/PW/슬랙ID. add the feature: change the name



## [471] need to match with figma about lead filter

- **Project**: Playbook

- **Status**: Close

- **URL**: https://www.notion.so/208b6d88d2cf8092b80ccc8251a98dfc

- **Description**: There are three filter options that need to match with Figma design



## [1430] Operation Hour

- **Project**: APP

- **Status**: Close

- **URL**: https://www.notion.so/2c9b6d88d2cf803b919ded38c319ac4e

- **Description**: How is it possible to 3time in there. We need to display just start time and end time



## [208] It's `Confirm Password` not `verify password`

- **Project**: Playbook

- **Status**: Close

- **URL**: https://www.notion.so/1ceb6d88d2cf80338c26fd2e44be2b46



## [908] Health status check answer submit is having error

- **Status**: Close

- **URL**: https://www.notion.so/24bb6d88d2cf80459c75f5253f7066a4



## [129] profile

- **Status**: Close

- **URL**: https://www.notion.so/1b0b6d88d2cf80969967c2a1c319bf77

- **Description**: 1. Profile update not properly
2. Contact Number is empty
~~3. When I select my gender it's showing korean, but profile showing english~~



## [976] Text/letter improvements (listing up here)

- **Status**: Close

- **URL**: https://www.notion.so/25cb6d88d2cf8091bb59c7f3a1caa8af



## [789] Unable to delete any user from the user table.

- **Status**: Close

- **URL**: https://www.notion.so/238b6d88d2cf806787d1c92c98c56f66



## [1307] Filter Missing

- **Status**: Close

- **URL**: https://www.notion.so/2a8b6d88d2cf80dcbb77fafca05e3289



## [1414] payment issue (1)

- **Status**: Close

- **URL**: https://www.notion.so/2c7b6d88d2cf80f69618c7f467156b4b

- **Description**: after payment password page, display gray screen. also not update the payment history page in app



## [1064] User log out error

- **Status**: Close

- **URL**: https://www.notion.so/298b6d88d2cf80318725cef357ec72dc

- **Description**: error while user logout
\{
\"message\": \"Unauthorized\",
\"errors_data\": \{
\"detail\": \"Unauthorized\",
\"code\": \"authentication_failed\"
\},
\"errors\": \{\},
\"status\": \"failure\",
\"status_code\": 401,
\"links\": \{\},
\"count\": 0,
\"total_pages\": 0,
\"data\": \[\]
\}



## [1120] Unknown looping issue

- **Status**: Close

- **URL**: https://www.notion.so/29cb6d88d2cf8045aa48eaf972a0ab87

- **Description**: Couldn't find the trigger, but there is a double loop of going back to "Abutment details" section when pressed Continue button on "Abutment design" section. The issue have happened for both Implant Crown and Implant Bridge



## [1310] error message is not correct

- **Status**: Close

- **URL**: https://www.notion.so/2a8b6d88d2cf80868ac4df1c9e89a623

- **Description**: message should be `You already created a blog with the same title`



## [816] can't login

- **Status**: Close

- **URL**: https://www.notion.so/23ab6d88d2cf80489dd2dbe6fb9af0fd



## [1760] My parents account linking is not working

- **Status**: Close

- **URL**: https://www.notion.so/92aebca79a0d497e83152a8fa3705111



## [1813] Profile info save having error

- **Status**: Close

- **URL**: https://www.notion.so/1046daca6e174b8ba91d3941d6203227



## [1212] Hold order error

- **Status**: Close

- **URL**: https://www.notion.so/2a1b6d88d2cf808497d1db58000cf4eb

- **Description**: For all Hold type



## [548] today assignment filter issue

- **Status**: Close

- **URL**: https://www.notion.so/217b6d88d2cf80a896ddf67b7b615830

- **Description**: today assignment leads not showing by the filter



## [679] Wish list counting is not working properly

- **Status**: Close

- **URL**: https://www.notion.so/223b6d88d2cf807bac7dc8f883f40c45



## [484] admin can't create the admin

- **Status**: Close

- **URL**: https://www.notion.so/20db6d88d2cf801fb98ee9d4c1247211



## [799] My info permission : checked 1st

- **Status**: Close

- **URL**: https://www.notion.so/239b6d88d2cf803188d6f738a477cc3d



## [1474] registere Phone number is not working

- **Status**: Close

- **URL**: https://www.notion.so/2d5b6d88d2cf80f3b56fc629e71ec0c1



## [604] task reminder is not working

- **Status**: Close

- **URL**: https://www.notion.so/21db6d88d2cf805da1ddc75612280052



## [727] Task time, input field is not working properly

- **Status**: Close

- **URL**: https://www.notion.so/229b6d88d2cf80d7bd0cf7323bd4e5a1



## [1160] Issue on order details after data is revised with back buttons.

- **Status**: Close

- **URL**: https://www.notion.so/29eb6d88d2cf8093972dc4ef9d03ac40

- **Description**: The data does not get override when we change the inputs to new record by using "back" button, and the order details are kept from original data. Both order details and price are paired with original data and doesn't get override with the new set of records.



## [856] Handle Duplicate Learning Diagnosis Submission

- **Status**: Close

- **URL**: https://www.notion.so/240b6d88d2cf80e4b358fde693d8a73d



## [1349] Change the word

- **Status**: Close

- **URL**: https://www.notion.so/2aeb6d88d2cf80b19f23db1684ff8b7a



## [1350] Change the word

- **Status**: Close

- **URL**: https://www.notion.so/2aeb6d88d2cf8000ad81ec25b659de06



## [1101] Bridge & Implant bridge order logic

- **Status**: Close

- **URL**: https://www.notion.so/29cb6d88d2cf808cb0e4c17de8cd30ba

- **Description**: Based on the order logic (Order Rules: row #10), IPS e.Max should be unable to select (display, but cannot click) with a crown range of 4 or more.



## [966] text capitalization needs correction

- **Status**: Close

- **URL**: https://www.notion.so/254b6d88d2cf8099b09eeefd93bcba96



## [2966] Full Preview Option Not Working

- **Status**: Close

- **URL**: https://www.notion.so/30cb6d88d2cf81b4bce2d65f1652d2d3

- **Dev Comment**: Fixed: DesignPreviewService now falls back to querying designPageRepository directly when pages are not loaded via relation. This resolves multi-page designs not rendering in full preview. Also registered DesignPage repository in DesignModule.



## [1438] operation hour

- **Status**: Close

- **URL**: https://www.notion.so/2cab6d88d2cf80eca8cfd8147003cb26



## [1296] Find feature

- **Status**: Close

- **URL**: https://www.notion.so/2a8b6d88d2cf80bd9ddcf3166e32e6ab



## [900] Validation error message needs minor modification

- **Status**: Close

- **URL**: https://www.notion.so/24bb6d88d2cf802680bbeee02c8953fe



## [1232] Missing Edit order on Drafted order

- **Status**: Close

- **URL**: https://www.notion.so/2a3b6d88d2cf80c7a62ad543df0b1fc1


