Relay - Secure Real-Time Chat App
---------------------------------
Relay is a full-stack chat application built to demonstrate clean architecture, real-time
communication, and secure data handling with envelope encryption.
It offers the simplicity of WhatsApp-style messaging while emphasizing backend security, modular
design, and scalability.
Features
--------
- Real-time 1-to-1 messaging (Socket.IO)
- User authentication (JWT or Supabase Auth)
- Message persistence (PostgreSQL)
- End-to-end transport security (TLS)
- Server-side Envelope Encryption for stored messages
- Key rotation with automatic re-wrapping
- Presence and typing indicators
- Modern responsive UI (React + TailwindCSS)
- TypeScript across frontend and backend
Tech Stack
----------
Frontend: React (Vite), TypeScript, TailwindCSS, Socket.IO client
Backend: Node.js, Fastify, TypeScript, Socket.IO, PostgreSQL
Database: PostgreSQL (via Supabase or Neon)
Auth: Supabase Auth (JWT)
Encryption: AES-256-GCM + HKDF envelope model
Deployment: Frontend -> Vercel, Backend -> Render/Fly.io
Monitoring: Sentry (errors), Prometheus (metrics)
Architecture Overview
---------------------
Frontend (React) <-> Socket.IO <-> Backend (Fastify + Node.js) <-> PostgreSQL <-> KMS/Vault
(KEK)
Encryption Model
----------------
Relay implements a custom envelope encryption system to secure stored chat data.
Flow:
1. Generate random 256-bit data key per message.
2. Encrypt plaintext with AES-256-GCM using that data key.
3. Encrypt (wrap) the data key with a long-term Key Encryption Key (KEK) stored in KMS or Vault.
4. Store ciphertext, IV, auth tag, wrapped key, and key version in the database.
5. On decryption, fetch KEK -> unwrap data key -> decrypt message.
Benefits:
- Unique key per message
- Cheap key rotation
- AES-GCM ensures integrity
- HKDF prevents key reuse
Database Schema
---------------
CREATE TABLE messages (
 id BIGSERIAL PRIMARY KEY,
 conversation_id UUID NOT NULL,
 sender_id UUID NOT NULL,
 ciphertext BYTEA NOT NULL,
 iv BYTEA NOT NULL,
 auth_tag BYTEA NOT NULL,
 wrapped_key BYTEA NOT NULL,
 key_version TEXT NOT NULL,
 algorithm TEXT DEFAULT 'AES-256-GCM-HKDF-v1',
 created_at TIMESTAMPTZ DEFAULT NOW()
);
Setup & Run
-----------
1. Clone repo
 git clone https://github.com/amitsinghrawat777/Relay
 cd relay
2. Install dependencies
 npm install
3. Create .env file and configure database, Supabase, and KEK
4. Run backend and frontend
 npm run dev:server
 npm run dev:client
Testing
-------
- Unit tests: crypto/envelope roundtrip (encrypt -> decrypt)
- Integration tests: message send/receive via socket
- Security tests: tamper detection, rewrap verification
Key Rotation
------------
1. Add new KEK (kek-v2) to KMS.
2. Run rewrap script to migrate wrapped keys.
3. Update system to use new key version.
Deployment
----------
Frontend: Vercel
Backend: Render or Fly.io
DB: Supabase / Neon
KMS: AWS or GCP
License
-------
MIT License © 2025 Amit
