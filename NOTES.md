# Notes

### Summary of Changes
- **SQL Operator Precedence**: Fixed missing parentheses in `TaskRepository.java`, `search_tasks.sql`, and `task_search_package.sql`. Previously, `AND` bound before `OR`, returning archived tasks and ignoring status filters on title matches.
- **Backend Latency & Robustness**: Removed artificial `Thread.sleep` in `TaskController.java`. Added safe handling for invalid `status` values (returns 400 Bad Request instead of 500) and clamped `page` and `pageSize` to prevent negative index errors.
- **Frontend Race Conditions & Debounce**: Added `useDebounce` hook (300ms) on search input to prevent network request spam. Added request cancellation and active-flag checks in `useTasks` to prevent out-of-order response overwrites. Fixed permanent "Loading..." lock when API calls fail by ensuring loading is reset on error. Reset pagination to page 1 on query/filter changes.

### What Was Not Changed & Why
- **In-Memory Pagination**: Kept `taskRepository.searchTasks()` returning a full list and slicing in Java. For an in-memory H2 database with small test data, rewriting to SQL `LIMIT`/`OFFSET` or JPA `Pageable` would increase diff size without immediate performance gain within the timebox.
- **Database Schema**: Left schema and data structures unchanged to keep the patch focused and ensure backward compatibility.

### Biggest Remaining Risk
The backend still loads all matching rows into memory before pagination. Under production volumes, this will cause memory pressure and degraded latency. Moving to true database-level pagination (`LIMIT`/`OFFSET`) and adding database indexes on `status` and `archived` should be the next priority.

### Tools and AI Used
Used Gemini to quickly inspect the codebase across layers, identify the SQL operator precedence flaw, draft the `useDebounce` hook, and refine the `AbortController` cancellation logic in `useTasks`.
