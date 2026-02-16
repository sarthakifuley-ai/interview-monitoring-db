# interview-monitoring-db
# AI Interview Monitoring System - Database Module

## 📋 Project Overview

AI system to detect cheating in online interviews using real-time video, audio, and behavior monitoring. Database stores all monitoring data and calculates risk scores.

**Tech Stack:** PostgreSQL 14.20  
**Database Team:** Kush Soni, Sarthaki, Chitranjan

---

## 📊 Database Schema

```
users → interview_sessions → object_detection_events (phones, extra persons)
                           → face_pose_events (head movements, looking away)
                           → audio_transcripts (speech to text)
                           → answer_evaluations (AI answer scoring)
```

**6 Tables:**
- `users` - User account info
- `interview_sessions` - Session tracking + risk scores
- `object_detection_events` - YOLOv8 object detections
- `face_pose_events` - OpenCV face tracking
- `audio_transcripts` - Whisper speech transcripts
- `answer_evaluations` - GPT/LLM answer scoring

**1 View:**
- `view_candidate_risk_summary` - Aggregated risk report

---

## 🔗 How to Integrate

### Backend Team (Omsrikar)

**Connection:**
```python
DATABASE_URL = "postgresql://user:pass@host:5432/interview_db"
```

**Main SQL Operations:**
```sql
-- Start interview
INSERT INTO interview_sessions (user_id, status) VALUES (?, 'IN_PROGRESS') RETURNING id;

-- Log AI module data
INSERT INTO object_detection_events (session_id, object_detected, confidence_score) VALUES (?, ?, ?);
INSERT INTO face_pose_events (session_id, event_type, severity_score) VALUES (?, ?, ?);
INSERT INTO audio_transcripts (session_id, text_content, is_multiple_speakers) VALUES (?, ?, ?);
INSERT INTO answer_evaluations (session_id, ai_relevance_score, ai_feedback) VALUES (?, ?, ?);

-- Update risk and end session
UPDATE interview_sessions SET total_risk_score = ? WHERE id = ?;
UPDATE interview_sessions SET end_time = NOW(), status = 'COMPLETED' WHERE id = ?;
```

---

### Frontend Team (Harsh, Rahul, Valarmathi)

**API Endpoints to Call:**
```
POST /api/interview/start              → Returns session_id
WS   /ws/video/{session_id}            → Stream video
WS   /ws/audio/{session_id}            → Stream audio
GET  /api/interview/{id}/risk          → Get current risk score
POST /api/interview/{id}/end           → End session
GET  /api/interview/{id}/report        → Get full report
```

---

### AI/ML Teams - Send Data in This Format

**YOLOv8** (Abdul, Sudarshana, Jeevan):
```json
{"session_id": 123, "object_detected": "cell phone", "confidence_score": 95.5}
```

**OpenCV** (Karlyn, Shreeya, Pearl, Rishikeshwar):
```json
{"session_id": 123, "event_type": "looking_away", "duration_ms": 3500, "severity_score": 3}
```

**Whisper** (Harsh, Omsrikar):
```json
{"session_id": 123, "text_content": "My answer is...", "is_multiple_speakers": false}
```

**GPT/LLM** (Dhwanil, Jaineel, Harsh):
```json
{"session_id": 123, "ai_relevance_score": 85.5, "ai_feedback": "Good understanding..."}
```

---

### Integration Team (Satyam, Gowthamkumar, Saksham, Shrinavasana)

**Setup Database:**
```bash
psql -U username -d interview_db -f interview_db_final_structure.sql
```

**Integration Flow:**
```
1. Frontend starts interview → Backend creates session_id
2. AI modules send data → Backend inserts into database
3. Backend calculates risk score → Updates interview_sessions table
4. Frontend requests report → Backend queries database and returns results
```

---

### Deployment Team (Faizal, Kush Soni)

**Docker Compose:**
```yaml
postgres:
  image: postgres:14.20
  environment:
    POSTGRES_DB: interview_db
    POSTGRES_USER: interview_user
    POSTGRES_PASSWORD: ${DB_PASSWORD}
  ports:
    - "5432:5432"
  volumes:
    - ./interview_db_final_structure.sql:/docker-entrypoint-initdb.d/init.sql
```

**AWS RDS:** PostgreSQL 14.20, db.t3.medium, 100GB SSD, SSL enforced

---

## 🚀 Quick Setup

```bash
# 1. Install PostgreSQL
sudo apt install postgresql-14
sudo systemctl start postgresql

# 2. Create database
sudo -u postgres psql
CREATE DATABASE interview_db;
CREATE USER interview_user WITH PASSWORD 'your_password';
GRANT ALL PRIVILEGES ON DATABASE interview_db TO interview_user;
\q

# 3. Import schema
psql -U interview_user -d interview_db -f interview_db_final_structure.sql

# 4. Verify
psql -U interview_user -d interview_db -c "\dt"
```

---

## 📝 Test It

```sql
-- Create user
INSERT INTO users (username, email, password_hash, full_name)
VALUES ('test', 'test@test.com', 'hash', 'Test User') RETURNING id;

-- Start session
INSERT INTO interview_sessions (user_id) VALUES (1) RETURNING id;

-- Log events
INSERT INTO object_detection_events (session_id, object_detected, confidence_score)
VALUES (1, 'cell phone', 95.5);

-- Check results
SELECT * FROM view_candidate_risk_summary WHERE session_id = 1;
```

---

## 🎯 Key Points

✅ 6 tables store all interview data  
✅ Each AI module inserts into its own table  
✅ Backend calculates final risk score  
✅ Pre-built view for quick reports  
✅ Indexed for fast queries

---

**File:** `interview_db_final_structure.sql` contains complete schema  
**Version:** 1.0 | **Date:** Jan 2026
