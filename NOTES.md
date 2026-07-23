# NOTES

## Approach

Before making any changes, I spent some time understanding how the application worked from both the frontend and backend. I ran the application locally, explored the available features, and then reviewed the code to identify issues that had the biggest impact on functionality and user experience.

Since the exercise mentions focusing on the highest-value bugs, I prioritized correctness, API behavior, performance, and frontend usability instead of making large architectural changes.

## Bugs Fixed

- Fixed incorrect SQL search logic caused by operator precedence.
- Removed an unnecessary delay from the task search endpoint.
- Fixed loading and error state handling in the React hook.
- Returned a proper HTTP 400 response for invalid status values.
- Reset pagination when search or filter values change.
- Added validation for invalid pagination parameters.

## Improvement

- Added a debounced search to reduce unnecessary API requests while typing.

## Assumptions

- Archived tasks should never appear in search results.
- Invalid query parameters should return a client error instead of a server error.
- Changing filters should always start from the first page of results.

## AI Usage

I used AI tools to help review the codebase, identify potential issues, and improve documentation. All fixes were reviewed, implemented, tested, and understood before being included in the final submission.