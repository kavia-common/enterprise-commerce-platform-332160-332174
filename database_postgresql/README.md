# database_postgresql

PostgreSQL database container for the enterprise commerce platform.

## Connection
The backend uses `DATABASE_URL` (see `backend_expressjs/.env`) to connect.

## Required schema for User Activity Log
Run the following SQL on your PostgreSQL instance:

```sql
CREATE EXTENSION IF NOT EXISTS pgcrypto;

CREATE TABLE IF NOT EXISTS user_activity_logs (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  created_at timestamptz NOT NULL DEFAULT now(),

  actor_user_id uuid NULL,
  actor_email text NULL,

  event_type text NOT NULL,
  resource_type text NOT NULL,
  resource_id text NULL,
  action text NOT NULL,

  metadata jsonb NULL,

  ip_address text NULL,
  user_agent text NULL,
  request_id text NULL
);

CREATE INDEX IF NOT EXISTS idx_user_activity_logs_created_at ON user_activity_logs(created_at DESC);
CREATE INDEX IF NOT EXISTS idx_user_activity_logs_event_type ON user_activity_logs(event_type);
CREATE INDEX IF NOT EXISTS idx_user_activity_logs_resource_type ON user_activity_logs(resource_type);
CREATE INDEX IF NOT EXISTS idx_user_activity_logs_actor_user_id ON user_activity_logs(actor_user_id);
```
