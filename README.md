# Course Folder Automation — Backend Module

Backend file-system logic for the **Course Folder Automation System**, a group internship project built to automate the creation and organization of academic course materials. This repository contains my individual contribution to the project — the folder-generation and record-management backend — developed as part of a 4-member team (Weeks 1–2).

> **Team project scope:** a PyQt6 desktop app + SQLite database that lets a user create a course folder structure and manage PDF records inside it. This repo covers only the backend logic layer (folder creation, record add/rename/move/delete) — not the GUI or database layers, which are owned by other team members.

## Status

| Week | Scope | Status |
|---|---|---|
| Week 1 | Course folder generation | ✅ Complete & tested |
| Week 2 | Record add / rename / move / delete | ✅ Complete & tested |
| Week 3 | Security module (hashing, audit logging) | 🔜 Planned |

## Contents

```
├── folder_generator.py        # Week 1 — course folder tree creation
├── test_folder_creation.py    # Week 1 — test suite (4 sample codes + edge cases)
├── record_manager.py          # Week 2 — add/rename/move/delete PDF records
├── test_record_manager.py     # Week 2 — test suite (10 test cases)
└── README.md
```

## Week 1 — Folder Generation

`folder_generator.py` takes a course code and a destination path, and creates the standard 7-category folder tree for that course:

```
<destination>/<course_code>/
    Assignments/
    Quizzes/
    Midterm/
    Final/
    Lab/
    Project/
    Presentations/
```

**Key function:**
```python
create_course_folders(course_code: str, destination: str, overwrite: bool = False) -> dict
```

**Edge cases handled:**
- Destination path doesn't exist → auto-created
- Course folder already exists → raises `CourseFolderExistsError` (unless `overwrite=True`)
- Invalid characters in course code → raises `InvalidCourseCodeError`

**Tested against:** `CS301`, `SE-205`, `MATH_101`, `CYB404` — 12/12 checks passed.

## Week 2 — Record Management

`record_manager.py` operates on individual PDF files inside an already-created course folder:

```python
add_record(source_file, course_root, category, db_insert_callback=None)
rename_record(file_path, new_name, db_update_callback=None)
move_record(file_path, new_category, db_update_callback=None)
delete_record(file_path, db_delete_callback=None)
```

**Design highlights:**
- **Disk-first, callback-driven:** every function performs its file-system operation first and only calls an optional callback (`db_insert_callback`, `db_update_callback`, `db_delete_callback`) after it succeeds — the database layer reflects disk state, never the reverse.
- **Collision handling:** duplicate filenames are auto-resolved with a numeric counter (`notes.pdf` → `notes(1).pdf`), not overwritten.
- **Security hook insertion points:** four points are marked inline (`SECURITY HOOK INSERTION POINT #1–4`) where Week 3's file-hashing and audit-logging will plug in without modifying existing logic.

**Edge cases handled:**
- Non-PDF file selected → `InvalidFileTypeError`
- Invalid/unknown category → `InvalidCategoryError`
- Rename/move/delete on a missing file → `RecordNotFoundError`

**Tested against:** add, duplicate-filename collision, invalid file type, invalid category, rename, move, delete, and three missing-file scenarios — 10/10 checks passed.

## Running the tests

No external dependencies — Python 3 standard library only.

```bash
python3 test_folder_creation.py
python3 test_record_manager.py
```

Both scripts clean up after themselves and print `[PASS]` / `[FAIL]` for every case.

## Integration points for the team

| Teammate | How this module connects to their work |
|---|---|
| **Ayesha (GUI)** | Calls `create_course_folders()` from the "Create Course" form, and `rename_record()` / `move_record()` / `delete_record()` from the right-click context menu. Catches the custom exceptions to show validation messages. |
| **Abdullah (Database)** | His Courses/Records table insert, update, and delete functions plug into the `db_insert_callback` / `db_update_callback` / `db_delete_callback` parameters. |
| **Amna (Lead/Review)** | The marked security hook insertion points are the exact spots identified for Week 3's hashing and audit-logging module. |

## Author

**Muhammad Aryan Tariq**
BS Cyber Security, Air University Islamabad
Roll No. 241552 — Section F24-B
