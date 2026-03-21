# Bug Report Documentation

## Bug 1: Database Connection Failure

### Observation  
While running the backend, the application crashed with an authentication error:  
`Access denied for user 'root'@'localhost'` 
The backend was trying to connect to MySQL without proper credentials.
<img width="940" height="294" alt="image" src="https://github.com/user-attachments/assets/14528e15-2a29-4746-971f-28067e6e0d67" />


### Root Cause  
The database connection string was missing the password.

### Fix  
Added the correct password in the database URI.

### Result  
Backend connected successfully and tables were created.

---

## Bug 2: Missing Cryptography Dependency

### Observation  
Backend crashed with error:  
`cryptography package is required for sha256_password authentication`
This occurred during MySQL authentication using PyMySQL.
<img width="940" height="232" alt="image" src="https://github.com/user-attachments/assets/e3027633-598a-4b2d-8b37-11e19e05e811" />


### Root Cause  
MySQL 8 uses a newer authentication method that requires the cryptography package.

### Fix  
Installed the dependency using pip.
<img width="940" height="290" alt="image" src="https://github.com/user-attachments/assets/023a8e6f-3328-40dd-900c-308de4cb41aa" />


### Result  
Authentication worked and backend started normally.

---


## Bug 3: Pagination Offset Miscalculation

### Observation  
API returned `total = 1` but `items = []`.
So, data existed in the database but was not being fetched correctly.
<img width="940" height="243" alt="image" src="https://github.com/user-attachments/assets/a52f8a07-ec27-4659-9863-36dceab4fdcb" />

<img width="640" height="273" alt="image" src="https://github.com/user-attachments/assets/a244e5cc-b050-4456-8c0e-8f79631f6de8" />


### Root Cause  
Wrong offset calculation in pagination.

### Fix  
Corrected offset:
```
offset = (page - 1) * per_page
```

### Result  
Expenses started appearing correctly.

---

## Bug 4: Incorrect Sum Calculation as String

### Observation  
Total sum was showing as a long concatenated string.
And values were treated as strings instead of numbers.
<img width="938" height="564" alt="image" src="https://github.com/user-attachments/assets/e63b256c-8373-4030-8242-bde8e587aa71" />


### Root Cause  
No type conversion before summing up.

### Fix  
Converted values using `Number()` before summing.

### Result  
Correct numeric sum displayed.

---

## Bug 5: Chart Not Rendering because of Wrong dataKey

### Observation  
Category chart was not showing any bars.
Backend returned:
```
{ "amt": number, "name": "Category" }
```
Frontend expected `value`.

### Root Cause  
Mismatch in field names.

### Fix  
Updated chart:
```
dataKey="amt"
```

### Result  
Chart started displaying correctly.

---

## Bug 6: Monthly Trend section Crash

### Observation  
Error:  
`Cannot read properties of undefined (reading 'map')`
Frontend expected a different response format than backend.
<img width="939" height="139" alt="image" src="https://github.com/user-attachments/assets/b3d818bf-d177-497d-8e68-32b0dceec6cb" />


### Root Cause  
Mismatch between:
- backend → `trend_rows`  
- frontend → `monthly_series`

### Fix  
Handled mapping safely:
```
(data.trend_rows || []).map(...)
```
If the data throws any error or is undefiend then an empty array [] is returned.

### Result  
Crash resolved and trend chart works.

---

## Bug 7: Deleted Expenses Still Visible in Expenses (paginated) section.

### Observation  
Deleted expenses were still showing in UI.
Soft delete was implemented (`is_deleted = 1`), but API was still returning those records.
<img width="939" height="463" alt="image" src="https://github.com/user-attachments/assets/73cb11e5-16aa-4a0e-ad2d-0afa9db03152" />


### Root Cause  
Missing filter in query.

### Fix  
Updated query:
```
Expense.query.filter(Expense.is_deleted == 0)
```
Also fixed total count.

### Result  
Deleted items no longer appear and pagination is correct.

---

## Enhancement: Backend Failures Handling 

### Observation  
During development, when the backend server stopped unexpectedly, the frontend started showing errors like failed API calls or blank sections without any clear message.

### Analysis  
The frontend was making API calls using fetch, but there was no proper error handling or user feedback when a request failed.

### Root Cause  
Missing error handling in API calls, which caused silent failures in the UI when the backend was not reachable.

### Suggestion / Improvement  
Add basic error handling using try-catch and check response status. Show a simple message (like an alert) when the backend is not responding.

Example:
```js
try {
  const res = await fetch(`${API}/expenses`);

  if (!res.ok) throw new Error("Server error");

  const data = await res.json();
} catch (e) {
  alert("Backend is not running or failed to respond.");
}
```

### Impact  
- Better user experience  
- Easier debugging during failures  
- Prevents confusion due to blank or broken UI
