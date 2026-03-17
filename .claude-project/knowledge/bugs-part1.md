## [3078] SMS service not set up — phone OTP does not work
- Dev Comment: Skipped - requires SMS provider business decision (e.g., Twilio, AWS SNS, Korean provider). No code changes needed until provider is selected.

## [3070] Coach signup form does not create an account
- Dev Comment: Already fixed in commit 7d790fe (2026-03-04). The handleSubmit was just calling navigate("/login") without any API call. Fixed by adding authService.register() with name, email, password, role=COACH. Also added loading state, error handling, and success message UI. Fix is on both ashadul.dev and dev branches.

## [3073] Admin patient table — Export button does nothing

## [3079] Remove unused counter code (counterSlice and Counter component)
- Dev Comment: Removed unused counter boilerplate code. Deleted counterSlice.ts and Counter.tsx, removed counterReducer from rootReducer.ts. Frontend build verified.

## [3072] Admin coach table — Export button does nothing

## [3071] Coach signup — license number and hospital code fields are never saved
- Dev Comment: Fixed full-stack pipeline: Added licenseNumber and hospitalCode columns to User entity, RegisterDto, auth.service register method, frontend RegisterRequest type, and coach signup form handleSubmit. Generated migration. Files: user.entity.ts, register.dto.ts, auth.service.ts, auth.d.ts, coach.tsx

## [3080] Coach and patient profile pages — user cannot edit their information

## [3075] Patient notifications — clicking a notification does not navigate anywhere

## [3077] Coach patient list — Sort button does nothing

## [3056] Dynamic input fields based on Role selection in 'Create New User' modal

## [3054] backbutton redirection issue

## [3053] The individual intensity doesn’t show the exercise record in admin dashboard.
- Dev Comment: Fixed: handleSimpleRating in exercise-detail.tsx was ignoring the rating param (prefixed _ as unused). Added RATING_TO_INTENSITY map (1=LOW/Easy, 2=MEDIUM/Normal, 3=HIGH/Difficult) and passed individualIntensity to logExercise.mutateAsync(). Admin panel ExerciseCell already displays intensity badges correctly - no backend changes needed. File: frontend/app/pages/patient/exercise-detail.tsx

## [3049] Layout breaks (Text wraps) when the current date is focused

## [3032] Inconsistent Session Bar Width in Calendar
- Dev Comment: Fixed by adding display: block to FullCalendar event objects. Timed events in month view were rendering as dot-events (content-width) instead of block-events (full cell width). File: frontend-dashboard/app/components/admin/calendar/CalendarContainer.tsx

## [3031] 'Deselect All' filter not working on Sessions Calendar
- Dev Comment: Fixed two bugs:
1. CalendarContainer.tsx: filteredMeetings returned ALL meetings when selectedCoachIds was empty. Changed to return [] instead.
2. home.tsx: API query fetched all meetings when no coaches selected. Added enabled: selectedCoachIds.length > 0 to skip the request.

Files modified:
- frontend-dashboard/app/components/admin/calendar/CalendarContainer.tsx
- frontend-dashboard/app/pages/admin/home.tsx

## [3030] Fix table header at the top while scrolling

## [3025] Wrong Redirection for "View Session Records”

## [3024] Add a checkmarker
- Dev Comment: Fixed 2 issues:
1. UI: Checkmark icon now displays inside checkbox when checked (fixed Tailwind CSS targeting)
2. Auto-login: Cookie now persists for 30 days when rememberMe is enabled (added maxAge to httpOnly cookie)

Files changed:
- frontend/app/pages/auth/login.tsx
- backend/src/core/interceptors/set-token.interceptor.ts

## [3023] Validation issue on the survey
- Dev Comment: Added onKeyDown handlers to both walking minutes and step count inputs to block non-numeric characters (e, E, +, -, .). File: frontend/app/pages/patient/survey.tsx

## [3022] Restrict access to future exercises and prevent editing past records
- Dev Comment: Implemented date-based access control:

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
- Dev Comment: Fixed getDaySummary() in exercise-day.service.ts to always validate totalCount against current active prescriptions instead of returning stale ExerciseDay records. Now if prescriptions are deactivated/removed, the count correctly reflects 0 and returns null.

## [3017] Date Dividers are Missing
- Dev Comment: Added date dividers to chat message list in both patient and coach views.

Files changed:
- NEW: frontend/app/components/chat/chat-date-divider.tsx (ChatDateDivider component + helpers)
- MOD: frontend/app/pages/patient/chat-detail.tsx (date divider rendering)
- MOD: frontend/app/pages/coach/chat-detail.tsx (date divider rendering)

Shows "Today", "Yesterday", or full date (e.g. 2026년 2월 25일) between messages from different days. Supports KO/EN i18n. Build verified.

## [3001] Indicator Mismatch with real data
- Dev Comment: Fixed: 3 bugs causing calendar dot indicators to not sync with real data.

1. Survey cache invalidation - useSubmitSurvey now invalidates all survey queries (surveyKeys.all) so calendar dots refresh after submission
2. Timezone mismatch - useSurveyStatusByDate now uses local timezone instead of UTC for today detection
3. Date parsing - Month view dot indicators now use string-based date extraction instead of fragile new Date() parsing

Files modified:
- frontend/app/services/httpServices/queries/useSurveys.ts
- frontend/app/pages/patient/home.tsx

## [2997] Fix data sync issues between Calendar and Zoom session info
- Dev Comment: ## Fix Implemented

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

## [2994] The word displayed is incorrect
- Dev Comment: Fix implemented: Changed exercise button text logic in patient home page. When today is selected and no exercises have been started (completedCount === 0), the button now shows '시작하기' instead of '이어서 하기'. '이어서 하기' is only shown when completedCount > 0 (user has started at least one exercise). File: frontend/app/pages/patient/home.tsx line 519.

## [2993] Exercise count is not updated on the home screen
- Dev Comment: Root Cause: Timezone mismatch in date parsing. new Date('YYYY-MM-DD') parses as UTC midnight, which shifts to the previous day in UTC+9 (Korea). This caused countActiveForPatient() to query the wrong date and return 0 prescriptions.

Fix: Created DateUtil.parseDateString() utility that parses date strings as local midnight. Applied across all exercise controllers and services.

Files Changed: date.util.ts (new), exercise-day.controller.ts, exercise-log.controller.ts, exercise-prescription.controller.ts, exercise-day.service.ts, exercise-log.service.ts

## [2982] Delete notification icon
- Dev Comment: Removed the notification bell icon button from the /patient/survey page header. The button had no click handler and was non-functional. Cleaned up the unused Bell import from lucide-react.

Files modified:
- frontend/app/pages/patient/survey.tsx

Commit: fix: remove notification icon from patient survey page

## [2980] Adjust the video height
- Dev Comment: Fixed exercise video layout to be full-width edge-to-edge. Removed padding, borders, and rounded corners from video container. Content below video retains proper padding. Container now uses flex-1 for proper height fill. Commit: 5e429d1

## [2978] Wrong Sender Label
- Dev Comment: Root Cause: Patient chat-detail.tsx identified sender role using ID comparison only (senderId === coachId). When admin sends messages from dashboard and is also the coach for that room, messages appear as Coach instead of Admin.

Fix:
1. Backend: Added sender.role to Socket.IO broadcasts (admin.service.ts + chat.gateway.ts)
2. Backend: Fixed getRoom endpoint to load patient/coach relations
3. Frontend: Updated ChatMessage.sender type to include role field
4. Frontend: chat-detail.tsx now checks sender.role === 1 (ADMIN) before coachId comparison

Files: chat-detail.tsx, chat.d.ts, admin.service.ts, chat.gateway.ts, chat.controller.ts

## [2977] Message Sorting Issue
- Dev Comment: Fixed: The backend chat message query (findByRoomId) was returning messages in DESC order (newest first) for pagination, but the frontend rendered the array as-is without reversing. This caused messages to appear newest-at-top instead of the standard oldest-at-top chat order.

Added .reverse() after the query so messages return in chronological order (oldest to newest). This fix applies to both patient and coach chat pages.

File changed: backend/src/modules/chat/repositories/chat-message.repository.ts

## [2976] Coach chat room is displayed as a seperated window
- Dev Comment: Removed redundant max-w-md mx-auto wrapper div from CoachChatDetail component. The coach layout already provides these container styles, so the double wrapping created a box-within-a-box effect with extra margins. Now uses React fragments like PatientChatDetail.

## [2975] Mismatch between the selected date and the clicking date.
- Dev Comment: Fixed timezone off-by-one bug in patient home calendar. handleDayClick used toISOString() which converts to UTC, shifting dates backward by 1 day in KST. Replaced with direct string formatting (YYYY-MM-DD). File: frontend/app/pages/patient/home.tsx (line 276-280)

## [2961] Dynamic Button Label based on Exercise Sequence
- Dev Comment: Changed button text in exercise-detail.tsx to show "다음 운동" (Next Exercise) for first/middle exercises and "운동 종료" (Finish) for the last exercise. Uses existing isLastExercise variable. Navigation (next/previous) was already working correctly.

## [2960] Reduce the vertical space

## [2951] Remove the hard-coding and display actual data
- Dev Comment: Removed hardcoded empty exercises array from exercise history page. Expanded day cards now lazy-load actual prescription data via GET /exercise-prescriptions/my?date=. Each exercise shows title, duration, sets, completion status, and difficulty. Also fixed stale React Query key for month navigation.

## [2940] Failed to set a coach
- Dev Comment: Root cause: Patient detail page sends fullName, phone, birth fields but UpdateUserDto only accepted username, email, role, coachId. Global ValidationPipe with forbidNonWhitelisted:true rejected the request.

Fix:
- Added fullName, phone, birth to UpdateUserDto
- Updated AdminService.updateUser() to persist new fields
- Added duplicate coach assignment guard
- Updated frontend UpdateUserRequest type
- Fixed patient-detail.tsx mutation typing

Files: update-user.dto.ts, admin.service.ts, admin.d.ts, patient-detail.tsx

## [2921] New users’s Days elapsed is invalid (minus value)
- Dev Comment: Fixed daysElapsed calculation — registrationDate was not zeroed to midnight like `now`, causing negative values for same-day users. Added `registrationDate.setHours(0, 0, 0, 0)` in both `mapUserWithAssignment()` and `getPatientsWithTodayPain()` in admin.service.ts.

## [2918] Reduce the vertical space
- Dev Comment: Reduced vertical spacing on exercise detail page (/patient/exercise/:id). Changes: (1) Video changed from square to 16:9 aspect ratio, (2) Reduced all vertical margins and padding, (3) Removed mt-auto from navigation buttons so they sit directly below the timer instead of being pushed to the bottom. All elements now fit on one mobile screen without scrolling.

## [2917] Exercise timer should reset for new exercises but save/resume for previous ones
- Dev Comment: Fixed per-exercise timer tracking in exercise-detail.tsx.

Changes:
1. Added useEffect on exercise ID change that resets timer to 00:00 for new exercises or restores saved time from DB logs for previously-completed exercises
2. Timer value (delta only) is now sent to backend via timeSpentSeconds in logExercise mutation
3. Backend accumulation logic was already correct - this was purely a frontend implementation gap

File modified: frontend/app/pages/patient/exercise-detail.tsx

## [2916] Remove the space in patient name
- Dev Comment: Fixed: Replaced firstName+lastName concatenation with fullName field in prescription-management.tsx. The page was manually joining firstName and lastName with a space, causing extra whitespace when one field was empty. Updated getPatientName(), getPrescribedByName(), and search filter to use fullName — consistent with all other dashboard pages.

## [2915] Expand selection exercise list 
- Dev Comment: Fixed: Increased min-h from 150px to 300px on the selected exercises section in PrescriptionEditorPanel.tsx (line 137) to show 5 exercises by default. File: frontend-dashboard/app/components/prescriptions/PrescriptionEditorPanel.tsx

## [2914] Timeline dot-lined overlap event item
- Dev Comment: Fixed in CalendarStyles.css:
1. Changed --fc-page-bg-color from transparent to var(--background) - restores 1px solid outline (box-shadow) around events that creates visible gaps between adjacent events in weekly/day views
2. Added solid background-color: var(--card) to .fc-timegrid-event - prevents dotted grid lines from showing through events
3. Fixed all hsl(var(--...)) patterns to var(--...) for Tailwind CSS v4 compatibility (oklch color values were being wrapped in hsl() which is invalid)

## [2904] keep only "Detailed" view

## [2900] Improve Calendar event layout and Fix line visibility

## [2899] User roles should only be COACH, PATIENT, ADMIN

## [2898] Chat room should be 3-person (patient + coach + admin), not 2-person

## [2360] Failed to create a new user

## [2359] Prescription record(history)

## [2358] Exercise library
- Dev Comment: Root cause: The seed script populated rep_time and unit columns but never populated default_reps and default_duration_seconds. The frontend reads defaultReps/defaultDurationSeconds to display reps/times, so all exercises showed "-". Additionally, exercise 8102 had Unit=분 (minutes) which the seed script did not handle, resulting in NULL unit.

Fix:
1. Migration 1770300000000: Fixes exercise 8102 (1분 → 60초), populates default_reps from rep_time for REPS exercises, populates default_duration_seconds from rep_time for SECONDS exercises.
2. Seed script updated: Added 분/minutes unit handling with seconds conversion. Added default_reps and default_duration_seconds to INSERT statement.

## [2338] User management

## [2337] Patient has two chatroom with the same coach
- Dev Comment: ### Fix Applied (Backend)

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

## [2334] Patient can not submit their survey

## [2332] Coach profile details
- Dev Comment: Added min-h-0 to the profile content wrapper div to fix flex overflow scrolling. The layout uses min-h-screen + overflow-hidden + flex-col, so without explicit min-h-0, the flex item couldn't shrink below content height and overflow-y-auto never activated. Also added no-scrollbar, z-10, and relative to match the pattern used by other working coach pages (patients, calendar, chat). Applied same fix to patient profile for consistency.

## [2333] User are unable to select another date
- Dev Comment: Fixed: Made patient home page (/patient) dynamic with date selection.

Changes:
1. Calendar dates are now clickable (week view + expanded). Selecting a date updates all content (survey, exercises, coaching session) for that date.
2. Added selected date visual indicator: blue ring in both views (distinct from today solid blue).
3. Week view now shows colored dots for all activities: green=exercise, yellow=survey, blue=session.
4. Fold button icon changed to proper upward arrow.
5. Date-aware labels adapt based on selected date. Past dates hide survey Start button.

Files: shared.d.ts, useSurveys.ts, calendar-grid.tsx, home.tsx
No backend changes.

## [2326] Need to prescriptions list page

## [2222] Exercise creation feature

## [1530] Use can see the history of submitted survey
- Dev Comment: Fixed: Survey history was limited to 10 entries with no pagination. Updated useSurveyHistory hook to support pagination parameters and return full response (surveys, total, page, limit). Added accumulating Load More button to patient survey history tab so users can view their complete survey history. The existing detail modal (click to view survey details) was already working correctly.

## [1531] admin log in failed message
- Description: There is no message for an invalid password.

## [1526] minus number
- Description: All number input fields accept positive integers from 0.

## [1525] Exercise prescription
- Description: The admin can prescribe the patient with 3-7 exercises. They need the search and filter exercise feature when prescribing.

## [1519] Admin profile avatar
- Description: A logout dropdown menu is required.

## [1517] Notification Icon
- Description: I don't think admin need the notification detail page

## [1516] Too many Role
- Description: The app has only one admin and two users (patient, coach)

## [1524] User creation
- Description: 1. Username 
2. login ID 
3. Password 
4. Password confirm 
5. Birth
6. phone *
7. Role

## [1515] Localization Issue on Login Button
- Description: For a brief moment, it appears in English rather than Korean.
- Dev Comment: Fixed: Added missing i18n key "auth.landing.loggingIn" to both ko.json ("로그인 중...") and en.json ("Logging in..."). The key was referenced in login.tsx but never defined in the translation files, causing i18next to display the raw key string instead of the translated text.

## [1508] All the component sizes
- Description: The header heights and profile size are different eachother. We need to bring all the sizes together. and don't need the red section if there is no exercise presciption

## [1507] Calendar gap(height) between the elements
- Description: It is too large.

## [1513] Coach profile page
- Description: There is no coach profile page now. Coach can't log out.

## [1506] My profile detail page
- Description: It should be directed to my profile page, not a dropdown.

## [1509] Padding sizes
- Description: Each has different padding.

## [1512] Quality ranges
- Description: The sleep quality assessment scale ranges from 1 to 10.

## [1510] Survey check effect
- Description: If the user chooses an option, the circle will be filled with dot.

## [3015] show password not matching in create account, confirm password as instruction or warning

## [3014] Onboarding information complete issue, loading in username

## [3013] SEARCH not connected with db

## [3012] No image showing in identify artwork.

## [3011] Object count issue in recognition picture

## [3010] No images showing on detected image. 

## [3009] Images ratio adjustment must be corrected. It’s bad now.

## [3008] Capture collection not created

## [3007] Password reset not works

## [3006] Button text overlay issue

## [2957] 
specs for the design elements used?i.e.) Typography Hierarchy, etc Font Size / Color RGB 

## [3133] Clickable button on Order Management & Add “Request Revision” “Awaiting Production” button instead of “On hold” and “Cancelled”

## [3132] Add more buttons from Dashboard as admin review below most frequently
“Drafted”, “Request Revision”, “Awaiting Production”

## [3130] Design review 3shape link insert and html insert design to be same.
- Description: 6

## [3129] When Reopen Order, “User ID” & and “Case Name” should be already inserted.

## [3128] Growth to be Last 30 days, not month-by-month

## [3127] Dashboard update to daily x-axis

## [3126] Need “Scanbody Type” section in Manage Content -> Order items (admin need to add more items)

## [3125] Add “ASC Abutment” in Abutment Type, and “Screwmentable” Retention Type. ASC Abutment is $150.

## [3124] Add “Veneer” in Restoration type, same decision tree as “Single Crown” → automatically add “Printed Model” in Additional Services
- Description: 4.1

## [3123] Added item not displayed in Order summary
- Description: 1.2

## [3122] Added item not displayed in invoice
- Description: 1.1

## [3121] Current chat functionally unstable
- Description:   • Does not display text after clicking box
  • Download button becomes smaller for a long file name
  • Text displays random chat area when clicked → should display the latest conversation
  • wrong chat display (Dr. Erin’s chat is Dr. Mark Valrose’s case)

[After several attempts, it displays the correct chat.]

## [3110] Page Reload Redirects to First Page
- Description: No matter which page I am currently on, if I reload that page, it always redirects me to the first page instead of staying on the current page.

## [3109] Image Generation Issue

## [3108] Error When Deleting Prompt After Design Generation (“Validation failed – uuid is expected”)
- Dev Comment: Added DELETE /design-editing/messages/:messageId endpoint with HTML rollback. Backend deletes user msg + assistant response + all subsequent msgs, rolls back page HTML to previous snapshot. Frontend trash icon on edit messages with confirmation dialog. Files: design-editing.service.ts, design-editing.controller.ts, designEditingService.ts, chatGenerationSlice.ts, UnifiedMessageList.tsx, chat-generate/index.tsx

## [3107] Generated Landing Page Appears Blank in Design View
- Dev Comment: Fixed: 1) Applied sanitizeInlineScripts to ALL iframe HTML (both preview and design modes) to prevent broken </script> tags in AI-generated HTML from corrupting the DOM and hiding page content. 2) Increased backend edit timeout from 80s to 120s to reduce timeout errors on large pages.

## [3106] Side Navbar issues among pages

## [3105] Generates Blank pages
- Dev Comment: Added global edit detection: when user mentions all pages/every page/globally etc in edit prompt, the backend now applies the same AI edit to all sibling pages automatically. Frontend re-fetches design pages after global edit to reflect changes.

## [3104] Navbar Should Be Hidden in Design Workspace
- Dev Comment: Fixed: Removed the app Header from the chat-generate workspace layout. The workspace already has its own back button and toolbar, giving users a distraction-free full-screen design environment.

## [3103] Prompt-Based Minor Edits Not Working (Edit Stream Connection Lost)

## [3102] Page Organization and Prototype Navigation Problems
- Dev Comment: Fixed 3 remaining issues: (1) Auto-group now triggers when restoring existing sessions, not just after fresh generation. (2) Anchor links (#section, #features, etc.) no longer blocked � properly skipped for in-page scrolling. (3) AI-generated links with spaces (e.g. "About Us") now normalized to hyphens before slug matching. Also improved: query string handling in catch-all regex, consistent lowercase slug normalization.

## [3100] Blank Page After Design Generation

## [3099] Mobile App Screen Not Using Full Width (Responsiveness Issue

## [3098] Default View Should Open in Preview Mode Instead of Edit Mode
- Dev Comment: Added Preview/Design view mode toggle in chat-generate page. Default is now Preview mode (no element selection). Users can switch to Design mode to click-edit elements. Also applied to my-designs detail page.

## [3097] Preview Container Not Optimized for 1440px Layout
- Dev Comment: Auto-fit zoom: preview now calculates optimal zoom based on container width to fill available space. Reduced padding from p-4 to p-2 in desktop mode. Removed hardcoded scale(0.6) from my-designs and GeneratingView, using full-width iframes instead.

## [3095] Inner Page Preview Not Working
- Dev Comment: Preview button always opened /designs/{id}/preview (no slug), showing the home page regardless of selected page. Fix: append current page slug to preview URL so inner pages open correctly.

## [3094] History Does Not Show Any Updates made by prompts
- Dev Comment: Version history was client-side only (Redux state), lost on page reload. Fix: in fetchEditMessagesByPage.fulfilled, rebuild version history from persisted htmlSnapshot fields in edit messages. Edit history now survives page refresh.

## [3093] Connection Error During Repeated Matching

## [3092] An unknown component appears 

## [3091] Inner Pages Generated Without Design
- Dev Comment: Code flow is correct (commit 3be832b fixed template-free sessions). Added retry mechanism: if generated page body text < 200 chars (empty/skeleton), regenerates once. This catches cases where AI produces minimal HTML.

## [3090] Bug: Captured artworks are not being sorted into the correct collection based on check-in status.
Expected: If user is checked in, capture should be added to the Location Capture Collection; if not checked in, capture should be added to the Untitled Capture Collection.

## [3089] Bug: When user captures an artwork photo, the Artwork Inference API is not being called, resulting in no AI recognition functionality during the Sprint 2 demo.
Expected: Artwork Inference API should be called immediately after photo capture, regardless of check-in status, and return artwork information on success or prompt manual entry on failure.

## [3082] Optimized Social Login Flow

## [3081] Attached Image Not Displaying in Chat
- Dev Comment: Images were sent to backend but never displayed in chat UI. Fix: added imageDataUrl to ChatMessage type, passed attachment data URL through Redux, and rendered image preview in MessageBubble component. Note: images show in current session only (not persisted to DB).

## [3051] Incorrect Output from PDF Analysis
- Dev Comment: Root cause: analyzeWithPrd() ignored uploaded PDF/image, determining pages from text only. A single landing page PDF was treated as a broad request, generating 20+ unrelated pages. Fix: pass imageData to PRD analysis + updated prompt to respect visual references.

## [3047] Chat History Not Fully Displayed
- Description: All we can see is our first prompt. when we reload the website no prompt showing except the first prompt
- Dev Comment: Additional fixes: (1) Added isRestoringSession loading state � shows skeleton loading UI while chat history is being fetched on page reload instead of the welcome prompt. (2) Generation complete status is derived from session state on restore (showDerivedCompleteMessage). (3) Messages are sorted chronologically after fetch. Previous fix already handles AI response display and FILE: marker stripping.

## [3046] Same template new page creation error
- Description: When we try to create new inner page its not creating, showing “Failed to add page”
- Dev Comment: Root cause: addPageToDesign required templateId, but template-free chat sessions have null templateId. Fix: use first existing page HTML as style reference when no template. Also fixed error message priority (backend message > generic Axios error).

## [3045] Second Prompt Not Being Processed
- Description: When I give the second or another prompt in the chat box, the second prompt doesn't appear in the chat section but it shows starting design generation but don’t do the design. 
- Dev Comment: Root cause (regression): The addPagePattern regex was too broad � it matched any prompt containing make/create/generate + any text + page. So edit prompts like "make this page look better" falsely triggered addNewPage instead of the edit path. Combined with UnifiedMessageList only rendering messages before gen-complete, user messages were invisible. Fix: (1) Tightened regex to require action word near page, (2) render post-gen messages, (3) backend uses BadRequestException, (4) await sendChatMessage before SSE, (5) guard for completed sessions without designId.

## [3044] Design Rendering Shows Raw Code
- Description: Some of the page is displaying raw code instead of rendering the proper design.
- Dev Comment: Root cause: AI-generated inline <script> tags contain literal </script> strings that break the browser parser, causing subsequent injected scripts to render as visible text. Fix: Added sanitizeInlineScripts() on both frontend (iframe-selection-script.ts) and backend (chat-generation.service.ts, gemini.service.ts) to escape nested </script> within script blocks. Also ensured missing </body></html> closing tags are appended.

## [3043] Version History Not Restoring Previous Design
- Dev Comment: Two fixes: 1) Version preview was wrapping full HTML docs inside <body>, creating malformed nested HTML. Now detects full docs and uses them directly. 2) Version restore now persists to backend via quickEdit API so it survives page refresh.

## [3042] Design Disappeared After Manual Text Change
- Dev Comment: Root cause: DOMParser re-serialization corrupts complex generated HTML. Fix: capture full page HTML directly from iframe DOM after text edit (removing overlay elements temporarily). Falls back to DOMParser path if iframe HTML unavailable.

## [3041] suggested workflow is:
- Dev Comment: Implemented all 3 features: (1) Template selector dialog with search and live preview in chatbox (Layout icon button, shown in chatting phase). (2) Attachment upload already existed (Paperclip button). (3) Image clipboard pasting - paste images directly into chatbox via Ctrl+V.

## [3034] Incomplete Design Output from Template for first time
- Description: When we try to generate using template initially it doesn’t provide any design. But when we reload the page we can see the design properly.
- Dev Comment: Fixed missing retry logic for fetchDesignBySession after generation completes. The chat SSE handler had retry logic but generation SSE, fallback polling, and safety net polling did not. Extracted shared fetchDesignWithRetry callback with exponential backoff (up to 3 attempts) and applied it to all 4 generation completion handlers. Design now loads reliably after generation without requiring a page reload.

## [3020] No Download HTML option after generationScreenshot
- Description: After a design is generated and shows 100% completion, there is no option available to Download the HTML code.
- Dev Comment: Download HTML/React buttons were gated behind isMultiPage condition. Removed the guard so buttons appear for all designs once a designId exists.

## [3019] template_id null constraint error when saving design

## [3018] Generated sections not rendering in design preview

## [1547] [High] Review design button inactive on request revision status
- Description: Customers can't easily check for new designs.

Tag: @Zihad

## [1546] [Urgent] Revision request status not visible in dashboard
- Description: Status not seen unless checking notification bar.

Tag: @Zihad

## [1545] [Ivory] Review design button inactive on request revision status
- Description: [Medium urgency] "Review design" button is inactive when status is on request revision. Customers need to consistently check for new designs unless notified via email.

## [1544] [Ivory] Revision request status not visible in dashboard
- Description: [Immediate attention needed] Revision request status not seen in the dashboard, easy to miss unless we check the notification bar. See attached images in Slack.

## [1543] Referral codes not following exm000 format for early users
- Description: Referral codes do not follow the agreed "exm000" format. Users with ID up to US00018 have referral codes that do not start with "exm".

Expected: All referral codes should follow exm + number format (e.g., exm001, exm045)
Actual: Early users (US00001-US00018) have different format

Reported by: Sun

## [1542] Orders not shown in User Details - Order Information table missing orders
- Description: Some orders do not appear in the Order Information section under User Details. This has been noticed with multiple user accounts.

Example: User US00045 (Dr. Patrick Loftus) shows "2 Total Orders" and "2 Monthly Orders" but only 1 order visible in table. Could be pagination/filter issue, Confirmed vs Draft tab issue, or data mismatch.

Reported by: Sun

## [1365] Chat alert issue
- Description: Not showing alerts on chat icon

## [1364] Add discount with $0
- Description: Change range for 0 to be valide input

## [1363] Design review comment notification
- Description: No notification or alerts, marked as read in Messages tab in Admin portal

## [1362] Not syncing on live server
- Description: status and notification not live synced           

## [1361] Merge order does not work
- Description: Production site

## [1346] Referral code issue
- Description: Cannot skip & submit or add referral code. Gives invalid phone number error

## [1345] Edit order issue
- Description: Duplicative Abutment Design fields in one page

## [1344] Status update for Refund request on Client end
- Description: Status update on Track this order

Client requested refund —> “Refund requested”

Admin refunded —> “Refunded”

Refund rejected —> “Refund declined”

## [1343] Refund item in order

## [1342] Tooth numbering issue

## [1341] Missing abutment calculation in Implant full arch

## [1340] No need for Interproximal contact in Implant full arch

## [1339] Glass ceramic should not be option in full arch
- Description: Only 4 options available, Monolithic zirconia, High translucent zirconia, Multi-layer zirconia, PMMA Temporary (or Temporary) 

## [1338] Back button & changing input doesn’t affect price estimate

## [1336] Status issue
- Description: All changes to Awaiting production after payment by client

## [1335] Calendly issue
- Description: No issues. might be server error

## [1334] New address not functioning
- Description: Does not show after adding a new addresss 

it’s working

## [1333] Leads to wrong page
- Description: Credit memo and invoice in the order detail do not match

## [1332] Continue button inactivation error during “Edit order”

## [1331] New function “Save order” in Dashboard button

## [1330] Price display after removing item during Edit order

## [1329] PMMA description shows without toogle on

## [1328] Pontic not displayed in Order Details

## [1327] Dropdown still gets cut out

## [1326] Viewable button not working

## [1325] After refunded by admin, should have “Refunded” status

## [1324] Back button need to go to “Order” page
- Description: Back button currently goes to where ever you came from, but should go to Order page as default.

## [1323] Order page loops back
- Description: When Order Edit & Implant Crown, order page Abutment Design loops back to Abutment Details so I have to go through the same two pages again. It loops once and it moves forward to Crown Details.

Abutment Design —> Abutment Details —> Abutment Design —> Abutment Details —> Crown Details…

## [1321] Proceed to Production button static

## [1304] Fp3 selected but no pink tissue showed

## [1303] Toggle off but not showing fp1, fp2, fp3. only fp3 showing.

## [1302] No retention type for full arch
- Description: for
Design Arch & provide abutments, Design and Mill Arch & provide abutments
show only Scanbody Type, MUA Abutment Type, Implant Type, Implant Connection

## [1301] Design Arch Only

Design Arch & Mill

Design Arch & Print

- Description: Only MUA user, MUA scanbody type selection will be here for Design Arch Only

Design Arch & Mill

Design Arch & Print


## [1300] MUA used, MUA scanbody type shouldn’t be display when 1. Design Arch & provide abutments
2. Design and Mill Arch & provide abutments selected
- Description: MUA used, MUA scanbody type shouldn’t be display when 1. Design Arch & provide abutments
2. Design and Mill Arch & provide abutments selected

Condition wrong - wrong options

## [1297] Admin can’t edit full arch draft

## [1295] Custom margin
- Description: Buccal - “Others - Please specify in Additional Instructions” - missing option

Lingual, Mesial, Distal  - missing after (-0.4)

## [1283] New registered login issue
- Description: Login goes to Registration, even when logo selected, still goes to Registration. Feels like loop created

## [1282] Edit bridge - success after select all tooth as crown but not continue after crown design
- Description: Order submitted - range 3 - crown 2 -pontic 1. 
Edit - select all 3 crown. - 0  pontic. 
- not continue after crown design while editing. but information updated. 

## [1281] Pink tissue - original
- Description: Pink tissue option - Original - missing

## [1280] Gingival shade option missing
- Description: c4, d2, d3, d4 not showing for Gingival shade

## [1279] Upload stl file - delete not working
- Description: After uploading, when delete STL file, skip for now doesn’t work. need to reupload stl to continue. 
For edit, when delete 
