# SnapClass Attendance System

SnapClass is a Streamlit attendance management application for teachers and students. It uses Supabase for persistent data, face recognition for student identification, and optional voice samples for voice-based attendance.

## Features

- Teacher registration and login.
- Create and manage subjects, sections, and shareable subject codes/QR links.
- Student profile registration using a face image, with an optional voice sample.
- Student subject enrollment by code or shared link, and attendance history.
- Teacher attendance capture from classroom photos or recorded audio, with review before saving.
- Teacher attendance summaries by subject and date/time.

## Requirements

- Python 3.10 or a compatible version for the dependencies in `requirements.txt`.
- A Supabase project and the tables/relationships used by the application.
- Webcam/audio input for capture features, or uploaded classroom photos.

The face recognition dependency includes native components (`dlib-bin` and `face_recognition_models`) and may require a platform-compatible Python environment.

## Setup

1. Clone or download this project and open a terminal in its root directory.
2. Create and activate a virtual environment:

   ```powershell
   py -m venv .venv
   .\.venv\Scripts\Activate.ps1
   ```

   On macOS/Linux:

   ```bash
   python3 -m venv .venv
   source .venv/bin/activate
   ```

3. Install dependencies:

   ```bash
   python -m pip install --upgrade pip
   pip install -r requirements.txt
   ```

4. Configure Supabase credentials in Streamlit secrets. Create `.streamlit/secrets.toml` locally (do not commit it) with:

   ```toml
   supabase_url = "https://YOUR_PROJECT.supabase.co"
   supabase_key = "YOUR_SUPABASE_KEY"
   ```

   The app reads these values in `src/database/config.py` as `st.secrets["supabase_url"]` and `st.secrets["supabase_key"]`. Use a key appropriate for the deployment and database policies.

5. Start the app from the project root:

   ```bash
   streamlit run app.py
   ```

Streamlit will print a local URL to open in your browser.

## Supabase data model

The code expects these Supabase tables and fields (at minimum):

- `teacher`: `teacher_id`, `username`, `password`, `name`.
- `student`: `student_id`, `name`, `face_embedding`, `voice_embedding`.
- `subject`: `subject_id`, `subject_code`, `name`, `section`, `teacher_id`.
- `subject_students`: `student_id`, `subject_id`, with relationships to `student` and `subject`.
- `attendace_logs` (spelling as used in the code): `student_id`, `subject_id`, `timestamp`, `is_present`, with relationships to `subject` and `student`.

Configure the corresponding foreign keys/relationships for the nested Supabase selects used by the app. No database migration or schema file is included in this repository, so the Supabase schema must be created separately to match these fields and relations.

## Project layout

```text
app.py                         Streamlit entry point and role routing
src/screens/                   Home, student, and teacher screens
src/component/                 Dialogs, subject cards, and reusable UI
src/pipelines/face_pipeline.py Face embeddings and attendance matching
src/pipelines/voice_pipeline.py Voice embeddings and speaker matching
src/database/config.py         Supabase client initialized from Streamlit secrets
src/database/db.py             Teacher, student, subject, and attendance operations
UI/                            Styles and image assets
.streamlit/                    Streamlit project configuration/notes
requirements.txt               Python dependencies
runtime.txt                    Deployment runtime hint
```

## Typical use

1. A teacher registers or logs in, creates a subject, and shares its subject code or join link.
2. A student registers by presenting a face image, optionally records a voice sample, and enrolls in a subject.
3. A teacher selects a subject, captures or uploads class photos or records audio, reviews the detected attendance, and confirms to save it.
4. Students view enrolled subjects and attendance counts; teachers view attendance summaries.

## Notes

- Face and voice identification depend on the quality of the captured images/audio and the embeddings stored for each student. Review results before saving.
- Face/voice embeddings and attendance records are sensitive personal data. Restrict access to the Supabase project, configure appropriate row-level security, and handle consent and retention according to your institution's policies.
- The database module imports Supabase during app startup, so valid Streamlit secrets must be configured before running the app.
- The shared subject link currently uses a fixed hosted app URL in `src/component/share_subject_dilog.py`; update it when deploying to a different URL.
- `.streamlit/` is ignored by Git in this project. Keep credentials out of version control and provide deployment secrets through the host's secure configuration.
