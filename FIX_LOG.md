# FIX LOG

## Bug 1: SQL WHERE Clause Operator Precedence

### Problem
While testing the search API, I noticed that archived tasks were still appearing in the results when the search term matched the task description. In some cases, the status filter was also ignored.

### How I Found It
I reviewed the query in `TaskRepository.java` and noticed that the `AND` and `OR` conditions were not grouped correctly. I confirmed the issue by testing the search API with different search terms and status filters.

### What I Changed
I added parentheses around the title and description search conditions so the query evaluates them together before applying the archived and status filters. I also updated the corresponding SQL reference files to keep them consistent.

### Why I Made This Change
The issue was caused by the SQL query logic rather than the application code. Grouping the conditions fixes the search behavior without changing the existing API or database structure.

### Validation
- Verified archived tasks are no longer returned.
- Verified status filtering works correctly with search.

### Files Modified
- `backend/src/main/java/com/internal/tasktracker/TaskRepository.java`
- `db/queries/search_tasks.sql`
- `db/oracle/task_search_package.sql`

---

## Bug 2: Removed Unnecessary Delay from Search API

### Problem
The search endpoint contained a `Thread.sleep()` call that delayed every request before executing the actual search. This made the application slower without adding any functional value.

### How I Found It
While reviewing `TaskController.java`, I noticed that every request calculated a delay based on the search query and paused execution using `Thread.sleep()`.

### What I Changed
I removed the delay calculation, the `Thread.sleep()` call, and the related logging. No business logic or API response was changed.

### Why I Made This Change
The delay was unnecessary and only slowed down every request. Removing it improves responsiveness while keeping the existing functionality unchanged.

### Validation
- Tested the search endpoint before and after the change.
- Verified search results remain the same.
- Confirmed responses are returned immediately.

### Files Modified
- `backend/src/main/java/com/internal/tasktracker/TaskController.java`

---
## Bug 3: Loading State Was Not Reset After API Failure

### Issue
If the task request failed because of a network or server error, the application stayed in the loading state and never displayed the error message. Previous errors also remained visible even after retrying.

### How I Found It
While reviewing `useTasks.js`, I noticed that `setLoading(false)` was only called when the request succeeded. I also observed that the previous error wasn't cleared before starting a new request. I confirmed this by stopping the backend and refreshing the page.

### Changes Made
I moved the loading cleanup into a `.finally()` block so it always runs, regardless of whether the request succeeds or fails. I also cleared the previous error before sending a new request.

### Reason for the Change
The loading state should always be reset after a request finishes. Using `.finally()` guarantees consistent behavior and improves the user experience when API requests fail.

### Validation
- Stopped the backend to simulate an API failure.
- Confirmed the loading spinner disappeared.
- Verified the error message was displayed correctly.
- Restarted the backend and confirmed the application recovered normally.

### Files Modified
- `frontend/src/hooks/useTasks.js`

---

## Bug 4: Invalid Status Parameter Returned HTTP 500

### Issue
Passing an invalid status value, such as `status=INVALID`, caused the API to return an HTTP 500 Internal Server Error instead of informing the client that the request was invalid.

### How I Found It
While testing the API with different query parameters, I tried an invalid status value and received a server error. Looking at `TaskController.java`, I found that `TaskStatus.valueOf()` was being called without handling invalid values.

### Changes Made
I wrapped the enum conversion in a `try-catch` block. If an invalid status is provided, the API now returns a clear HTTP 400 Bad Request response with an appropriate error message.

### Reason for the Change
Invalid user input should be treated as a client error rather than causing the server to fail. Returning HTTP 400 makes the API behavior more predictable and prevents unnecessary server errors.

### Validation
- Tested the endpoint with valid status values.
- Tested with `status=INVALID`.
- Confirmed the API now returns HTTP 400 with a meaningful error message.

### Files Modified
- `backend/src/main/java/com/internal/tasktracker/TaskController.java`

---

## Bug 5: Pagination Was Not Reset When Search or Filter Changed

### Issue
When I was on page 2 or later and changed the search text or status filter, the application kept the same page number. If the new result set had fewer pages, the table showed "No tasks found" even though matching tasks existed.

### How I Found It
I tested the application by navigating to page 3 and then searching for a specific task. Instead of showing the matching results, the table displayed an empty state because the current page was no longer valid for the new search.

### Changes Made
I added wrapper functions for the search and status change handlers. Whenever the search query or status filter changes, the current page is reset to page 1 before fetching new data.

### Reason for the Change
Changing the search criteria should always start from the first page of results. This prevents invalid page requests and provides a better user experience.

### Validation
- Moved to page 3.
- Changed the search query.
- Changed the status filter.
- Confirmed the page resets to 1 and matching results are displayed correctly.

### Files Modified
- `frontend/src/App.jsx`

---

## Bug 6: Added Validation for Invalid Pagination Parameters

### Issue
The API accepted invalid values such as negative page numbers or a page size of zero. These values could cause invalid list indexing and result in an internal server error.

### How I Found It
While testing the API with different query parameters, I tried values like `page=-1` and `pageSize=0`. This caused the backend to fail because invalid indexes were being calculated.

### Changes Made
I added validation to ensure the page number is always at least 1 and the page size stays within a safe range. The controller now uses the validated values before processing the request.

### Reason for the Change
The API should handle invalid client input safely instead of failing with a server error. Validating the parameters makes the endpoint more reliable and prevents unexpected runtime errors.

### Validation
- Tested valid page and page size values.
- Tested negative and zero values.
- Confirmed the endpoint now handles invalid input safely without returning an internal server error.

### Files Modified
- `backend/src/main/java/com/internal/tasktracker/TaskController.java`

---

## Improvement 1: Added Debounced Search Requests

### Improvement
Every time a user typed a character in the search box, the application immediately sent a new API request. This resulted in multiple requests being made while the user was still typing.

### How I Found It
While testing the search feature, I monitored the browser's Network tab and noticed that every keystroke triggered a separate API call. For example, typing a word like "pagination" generated multiple requests in quick succession.

### Changes Made
I created a reusable `useDebounce` hook and updated the application to use the debounced search value when fetching tasks. The search input remains responsive, but API requests are only sent after the user pauses typing.

### Reason for the Change
This reduces unnecessary API calls, improves the search experience, and lowers the load on the backend without changing the existing functionality.

### Validation
- Typed quickly in the search box.
- Confirmed the input remained responsive.
- Verified that only one API request was sent after typing stopped.
- Confirmed search results remained unchanged.

### Files Modified
- `frontend/src/hooks/useDebounce.js`
- `frontend/src/App.jsx`