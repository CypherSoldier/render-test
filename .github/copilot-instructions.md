# Copilot Instructions - Part3 Phonebook API

## Project Overview
Part3 is a Node.js backend project featuring two separate Express REST APIs running on port 3001:
- **Notes API** ([index.js](../index.js)): Simple CRUD for text notes with importance flag
- **Persons/Phonebook API** ([main.js](../main.js)): Contact management with name and phone number

Both APIs use in-memory arrays for data storage (no database) and share identical architectural patterns.

## Architecture & Data Structures

### Shared Patterns
- **In-Memory Storage**: Data persists only during server runtime (arrays: `notes[]` and `persons[]`)
- **ID Generation**: String-based IDs; notes use incremental logic, persons use random generation
- **Response Format**: All responses are JSON objects or arrays; delete operations return 204 No Content
- **Validation**: Request body validation occurs before ID generation; missing required fields reject with 400 status

### Notes API Structure
```javascript
const note = {
  id: "1",              // String, generated via generateId()
  content: "...",       // Required field
  important: false      // Boolean, defaults to false if omitted
}
```

### Persons API Structure
```javascript
const person = {
  id: "1",              // String, random 1-100 + 1
  name: "...",          // Required; must be unique (checked via Array.find)
  number: "..."         // Required phone format
}
```

## Key Development Workflows

### Running the Server
```bash
npm run dev      # Runs with --watch flag for auto-reload on file changes
npm start        # Same as dev (watch mode enabled in package.json)
```

### Testing Endpoints
Use VS Code REST Client extension with files in [requests/](../requests/) directory:
- [creating_new_note.rest](../requests/creating_new_note.rest)
- [creating_new_person.rest](../requests/creating_new_person.rest)
- [get_all_notes.rest](../requests/get_all_notes.rest)

Endpoints return on http://localhost:3001

## Critical Implementation Notes

### Known Issues to Fix
1. **main.js, DELETE endpoint**: Uses undefined `response` variable (should be `res`)
2. **index.js, GET /:id endpoint**: Returns note twice - missing early return after 404 check
3. **ID Generation Inconsistency**: Notes use `Math.max()` logic; persons use random - should standardize

### Validation Patterns
- Check for missing required fields **before** generating IDs
- Persons API: Validate uniqueness of names using `Array.find(p => p.name === body.name)`
- Return 400 with error object for validation failures: `{ error: 'description' }`

### Data Mutation
Both APIs directly reassign array variables:
```javascript
notes = notes.concat(newNote)
persons = persons.filter(p => p.id !== id)
```

## Common Modifications
- **Adding a new endpoint**: Follow Express pattern `app.method('/path/:params', (req, res) => { ... })`
- **Adding fields to data model**: Update structure in initial array AND in creation endpoint
- **Changing validation rules**: Modify checks in POST handlers before `generateId()` call
- **Adding logging**: Use Morgan middleware already in dependencies (currently unused in code)

## Tech Stack
- **Framework**: Express ^5.2.1
- **Logging Middleware**: Morgan ^1.10.1 (installed but not configured)
- **Runtime**: Node.js with --watch for development
- **Testing**: Manual HTTP testing via .rest files

## Notes for AI Agents
- When extending APIs, maintain consistency with existing patterns (one array-based store per API, string IDs)
- Always validate request bodies before mutations
- Return appropriate HTTP status codes: 200/201 for success, 400 for validation, 404 for not found, 204 for delete
- Test changes using provided .rest files or by manually creating similar requests
