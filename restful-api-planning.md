# RESTful API Planning Document

## 1. API Selection

**Selected API:** Open Library Search API  
**Public API directory:** https://github.com/public-apis/public-apis  
**Documentation:** https://openlibrary.org/dev/docs/api/search

I will design a client-side book search application that retrieves book information from Open Library. This API is appropriate because its public search endpoint returns structured JSON and supports queries without requiring a user account. Users will be able to search for books by title and view results such as title, author, publication year, and availability of cover images.

The design follows REST principles by identifying books and search results through resource URLs, using the standard **GET** method for retrieval, and keeping requests **stateless**: each search request contains the query needed to process it, rather than depending on a previous request. A separate proposed Python backend will accept book recommendations; that backend is part of the plan, not an existing Open Library endpoint.

## 2. Endpoint Mapping

| Service | Method and endpoint | Purpose | Data |
| --- | --- | --- | --- |
| Open Library | `GET https://openlibrary.org/search.json?title={query}&limit=10` | Search for books by title | JSON response containing `docs` with book metadata |
| Proposed Python backend | `POST /api/recommendations` | Submit a user's book recommendation | Accepts JSON with `book_title`, `reviewer_name`, `rating`, and `comments`; returns a confirmation and created ID |
| Proposed Python backend | `GET /api/recommendations` | Retrieve submitted recommendations | Returns a JSON array of recommendations |

Each endpoint represents an identifiable resource or collection. GET requests read data without modifying it, while POST creates a recommendation. The planned backend will treat each request independently, use JSON request and response bodies, and return appropriate HTTP status codes such as `200`, `201`, `400`, and `500`. The `/api/recommendations` endpoints are **proposed**, not implemented or provided by Open Library.

## 3. Data Flow Diagram (Textual)

**User enters book title → Client validates search → Loading indicator appears → JavaScript `fetch()` sends asynchronous GET → Response is checked → JSON is parsed → Results are mapped into cards or table rows → Conditional styles are applied → Loading indicator disappears.**

The browser will call `fetch('https://openlibrary.org/search.json?title=' + encodeURIComponent(query) + '&limit=10', { headers: { Accept: 'application/json' } })`. `encodeURIComponent` safely places the user's search text into the URL. The client will use `async`/`await`, check `response.ok`, and call `response.json()` to read the returned JSON. An `isLoading` state will display “Searching…” and disable repeated submissions during the request. A `try/catch/finally` structure will display a friendly error for network or HTTP failures and always clear the loading state. An empty `docs` array will display “No books found.”

The UI will map `data.docs` into book cards displaying the title, first author (when available), and first publication year. Missing values will be shown as “Unknown.” Conditional styling will assign a `.classic` CSS class to books with a first publication year before 1950 and a `.recent` class to books published in or after 2000; other books will use `.standard`. This makes results easier to scan without changing the underlying data.

## 4. Form Handling

The interface will have two forms:

**Search form**
- `title_query`: required text, trimmed, 2–100 characters. This value becomes a URL-encoded query parameter in the public GET request; no JSON body is sent with GET.

**Recommendation form (for the proposed Python backend)**
- `book_title`: required text, trimmed, 2–150 characters.
- `reviewer_name`: required text, trimmed, 2–80 characters.
- `rating`: required integer from 1 through 5.
- `comments`: optional text, maximum 500 characters.

Client-side JavaScript will validate inputs and display field-specific messages before submission. Valid recommendation values will be transformed into a JSON object, for example:

```json
{
  "book_title": "Pride and Prejudice",
  "reviewer_name": "Sample Reader",
  "rating": 5,
  "comments": "A memorable classic."
}
```

The planned request is `fetch('/api/recommendations', { method: 'POST', headers: { 'Content-Type': 'application/json', Accept: 'application/json' }, body: JSON.stringify(payload) })`. The Python backend would validate the same constraints again and return `201 Created` with the new record, or `400 Bad Request` for invalid input. The client will show success or error feedback. Only the contract is being designed for this assignment; the backend is not being built.

## 5. Technology Stack

- **HTML5:** semantic page layout, search form, recommendation form, and result containers.
- **CSS3:** responsive book cards, loading and error messages, and `.classic`, `.recent`, and `.standard` classes.
- **JavaScript (vanilla):** Fetch API, `async`/`await`, input validation, JSON parsing, and DOM updates.
- **Python + Flask (planned):** optional future REST backend for `/api/recommendations`.
- **VS Code:** writing and organizing project files.
- **Browser Developer Tools:** inspecting network requests, HTTP status codes, console errors, and rendering.
- **curl or Postman:** testing the planned backend's GET and POST requests once implemented.
- **Git and GitHub:** version control, documentation, and repository hosting.

## 6. Implementation Steps

1. **Set up the project:** Create a GitHub repository and plan files for HTML, CSS, JavaScript, and documentation. Record the public API endpoint and response fields.
2. **Implement asynchronous GET:** Build the book search form, validate its input, call Open Library using `fetch()`, and handle loading, HTTP failures, network errors, and empty results.
3. **Render dynamic results:** Convert the returned `docs` array into book cards or table rows. Show fallback labels for missing author or year data.
4. **Add form controls and validation:** Create the recommendation form and enforce required fields, character limits, and the 1–5 rating range.
5. **Plan backend integration:** Define the JSON POST contract for `/api/recommendations`, response codes, and server-side validation; implementation is outside the scope of this planning assignment.
6. **Apply conditional styling:** Highlight classic and recent books using CSS classes determined by publication year. Check responsive behavior and accessibility.
7. **Test and review:** Test valid and empty searches, special characters, unavailable API responses, missing metadata, invalid form inputs, and successful JSON serialization. Review REST statelessness and documentation.
8. **Document and submit:** Commit `restful-api-planning.md` to the new public GitHub repository and submit the repository URL.

## References

- Open Library. (n.d.). *Search API*. https://openlibrary.org/dev/docs/api/search
- Public APIs. (n.d.). *A collective list of free APIs*. GitHub. https://github.com/public-apis/public-apis
