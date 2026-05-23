# Plan B: Aggressive Path - Technical Implementation Plan

**Tauri 2.0 + Contacts Only | 3 weeks | 3x Claude Code instances | $75/month**

---

## 1. Technology Stack

### Rust Backend (Tauri)

| Package (Crate) | Version | Purpose | Category |
|-----------------|---------|---------|----------|
| tauri | 2.x | Desktop framework (Rust backend + OS webview) | Framework |
| tauri-build | 2.x | Build scripts for Tauri | Build |
| tauri-plugin-sql | 2.x | SQLite plugin for Tauri | Database |
| rusqlite | ^0.32 | Direct SQLite access with FTS5 support | Database |
| serde | ^1.0 | Serialization framework | Utility |
| serde_json | ^1.0 | JSON serialization/deserialization | Utility |
| ulid | ^1.1 | Sortable unique ID generation | Utility |
| chrono | ^0.4 | Date/time handling | Utility |
| csv | ^1.3 | Fast CSV parsing in Rust | Utility |
| tauri-plugin-dialog | 2.x | Native file open/save dialogs | Plugin |
| tauri-plugin-fs | 2.x | File system access | Plugin |
| tauri-plugin-updater | 2.x | Auto-update support | Plugin |
| tauri-plugin-shell | 2.x | Open URLs in default browser | Plugin |
| thiserror | ^2.0 | Error type derivation | Utility |
| tokio | ^1.41 | Async runtime (used by Tauri) | Runtime |

### React Frontend

| Package | Version | Purpose | Category |
|---------|---------|---------|----------|
| react | ^19.0 | UI library | UI |
| react-dom | ^19.0 | React DOM renderer | UI |
| typescript | ^5.6 | Type safety | Language |
| vite | ^6.0 | Frontend bundler + HMR | Build |
| @vitejs/plugin-react | ^4.3 | Vite React integration | Build |
| @tauri-apps/api | ^2.x | Tauri frontend API (invoke, events) | Framework |
| @tauri-apps/plugin-dialog | ^2.x | Dialog API for frontend | Plugin |
| @tauri-apps/plugin-fs | ^2.x | File system API for frontend | Plugin |
| tailwindcss | ^4.0 | Utility-first CSS | Styling |
| zustand | ^5.0 | Lightweight state management | State |
| zod | ^3.24 | Frontend validation | Validation |
| cmdk | ^1.0 | Command palette | UI |
| @tanstack/react-virtual | ^3.11 | Virtual scrolling for large lists | UI |
| react-focus-lock | ^2.13 | Focus trapping for modals | UI |
| lucide-react | ^0.468 | Icon library | UI |
| date-fns | ^4.1 | Date formatting | Utility |

### Testing

| Package | Version | Purpose | Category |
|---------|---------|---------|----------|
| vitest | ^2.1 | Frontend unit/component testing | Testing |
| @testing-library/react | ^16.1 | React component testing | Testing |
| playwright | ^1.49 | E2E testing (WebDriver mode for Tauri) | Testing |
| cargo test | (built-in) | Rust backend unit testing | Testing |

### Developer Tooling

| Package | Version | Purpose | Category |
|---------|---------|---------|----------|
| eslint | ^9.16 | Frontend linting | Tooling |
| prettier | ^3.4 | Code formatting | Tooling |
| clippy | (built-in) | Rust linting | Tooling |
| rustfmt | (built-in) | Rust formatting | Tooling |
| @tauri-apps/cli | ^2.x | Tauri dev server + build | Tooling |

---

## 2. Project Structure

```
local-crm/
├── src-tauri/
│   ├── src/
│   │   ├── main.rs              # Tauri entry point
│   │   ├── lib.rs               # Command registration + app setup
│   │   ├── error.rs             # Custom error types (thiserror)
│   │   ├── state.rs             # AppState (database connection)
│   │   ├── db/
│   │   │   ├── mod.rs           # Module exports
│   │   │   ├── schema.rs       # SQLite schema creation + migrations
│   │   │   ├── contacts.rs     # Contact CRUD queries
│   │   │   ├── search.rs       # FTS5 search queries
│   │   │   ├── tags.rs         # Tag CRUD + junction queries
│   │   │   ├── settings.rs     # Settings get/set queries
│   │   │   └── shortcuts.rs    # User shortcut queries
│   │   ├── commands/
│   │   │   ├── mod.rs           # Module exports
│   │   │   ├── contacts.rs     # #[tauri::command] for contacts
│   │   │   ├── search.rs       # #[tauri::command] for search
│   │   │   ├── tags.rs         # #[tauri::command] for tags
│   │   │   ├── import_export.rs # #[tauri::command] for CSV
│   │   │   ├── settings.rs     # #[tauri::command] for settings
│   │   │   └── shortcuts.rs    # #[tauri::command] for shortcuts
│   │   └── models/
│   │       ├── mod.rs           # Module exports
│   │       ├── contact.rs      # Contact, CreateContact, ContactFilter structs
│   │       ├── tag.rs          # Tag struct
│   │       ├── custom_field.rs # CustomFieldDefinition struct
│   │       └── settings.rs     # Settings types
│   ├── Cargo.toml               # Rust dependencies
│   ├── tauri.conf.json          # Tauri app configuration
│   ├── capabilities/
│   │   └── default.json         # Permission capabilities
│   ├── icons/                   # App icons (all sizes)
│   └── build.rs                 # Tauri build script
├── src/
│   ├── App.tsx                  # Root component
│   ├── main.tsx                 # React entry point
│   ├── index.css                # Tailwind imports + globals
│   ├── components/
│   │   ├── layout/
│   │   │   ├── Sidebar.tsx      # Navigation (contacts, settings)
│   │   │   ├── TopBar.tsx       # Search + actions
│   │   │   └── KeyboardHelp.tsx # Shortcut reference modal
│   │   ├── contacts/
│   │   │   ├── ContactList.tsx  # Virtual scrolling list
│   │   │   ├── ContactDetail.tsx # Contact view with actions
│   │   │   ├── ContactForm.tsx  # Create/edit form
│   │   │   └── ContactRow.tsx   # Single row in list
│   │   ├── search/
│   │   │   ├── SearchBar.tsx    # Global search
│   │   │   └── SearchResults.tsx # Result list
│   │   ├── shared/
│   │   │   ├── CommandPalette.tsx # cmdk command palette
│   │   │   ├── TagInput.tsx      # Tag selector
│   │   │   ├── CustomFieldRenderer.tsx # Custom field by type
│   │   │   └── VirtualList.tsx   # Virtual scroll wrapper
│   │   └── import/
│   │       ├── CSVImport.tsx    # Import wizard
│   │       └── FieldMapper.tsx  # Column mapping
│   ├── hooks/
│   │   ├── useKeyboard.ts      # Shortcut registration
│   │   ├── useContacts.ts      # Contact operations
│   │   ├── useSearch.ts        # Search with debounce
│   │   └── useCommandPalette.ts
│   ├── stores/
│   │   ├── contactStore.ts     # Contact state
│   │   ├── uiStore.ts          # UI state
│   │   └── shortcutStore.ts    # Shortcut state
│   ├── lib/
│   │   ├── tauri.ts            # Typed invoke wrapper
│   │   ├── keyboard.ts         # Shortcut engine
│   │   ├── validators.ts       # Zod schemas
│   │   └── constants.ts        # App constants
│   └── types/
│       ├── contact.ts          # Contact types
│       └── custom-field.ts     # Custom field types
├── tests/
│   ├── unit/                   # Frontend unit tests
│   ├── component/              # React component tests
│   └── e2e/                    # Playwright E2E tests
├── package.json
├── vite.config.ts
├── tsconfig.json
└── tailwind.config.ts
```

**Directory roles:**

- **src-tauri/** - The Rust backend. Contains all SQLite operations, business logic, and Tauri command definitions. This replaces Electron's main process.
- **src-tauri/src/db/** - Pure database functions. Each file handles one domain (contacts, tags, search). No Tauri dependencies here, just rusqlite queries. Easy to unit test.
- **src-tauri/src/commands/** - Tauri command layer. Thin wrappers that extract state, call db functions, and return results. Decorated with `#[tauri::command]`.
- **src-tauri/src/models/** - Rust structs with Serde derive macros. Shared between db and command layers. Auto-serialize to/from JSON for the frontend.
- **src/** - React frontend. Identical structure to Plan A minus deals components. Communicates with Rust via `invoke()`.

---

## 3. Database Schema

```sql
-- Enable WAL mode for concurrent reads
PRAGMA journal_mode = WAL;
PRAGMA foreign_keys = ON;

----------------------------------------------
-- CONTACTS
----------------------------------------------
CREATE TABLE contacts (
    id          TEXT PRIMARY KEY,          -- ULID
    name        TEXT NOT NULL,
    email       TEXT,
    phone       TEXT,
    company     TEXT,
    title       TEXT,                      -- Job title
    notes       TEXT,
    custom_fields TEXT DEFAULT '{}',       -- JSON object
    created_at  TEXT NOT NULL DEFAULT (datetime('now')),
    updated_at  TEXT NOT NULL DEFAULT (datetime('now')),
    archived_at TEXT                       -- NULL = active
);

CREATE INDEX idx_contacts_email ON contacts(email);
CREATE INDEX idx_contacts_company ON contacts(company);
CREATE INDEX idx_contacts_created ON contacts(created_at);
CREATE INDEX idx_contacts_archived ON contacts(archived_at);
CREATE INDEX idx_contacts_name ON contacts(name);

----------------------------------------------
-- FULL-TEXT SEARCH (FTS5)
----------------------------------------------
CREATE VIRTUAL TABLE contacts_fts USING fts5(
    name,
    email,
    company,
    phone,
    notes,
    content='contacts',
    content_rowid='rowid',
    tokenize='porter unicode61'
);

-- Sync triggers
CREATE TRIGGER contacts_ai AFTER INSERT ON contacts BEGIN
    INSERT INTO contacts_fts(rowid, name, email, company, phone, notes)
    VALUES (new.rowid, new.name, new.email, new.company, new.phone, new.notes);
END;

CREATE TRIGGER contacts_ad AFTER DELETE ON contacts BEGIN
    INSERT INTO contacts_fts(contacts_fts, rowid, name, email, company, phone, notes)
    VALUES ('delete', old.rowid, old.name, old.email, old.company, old.phone, old.notes);
END;

CREATE TRIGGER contacts_au AFTER UPDATE ON contacts BEGIN
    INSERT INTO contacts_fts(contacts_fts, rowid, name, email, company, phone, notes)
    VALUES ('delete', old.rowid, old.name, old.email, old.company, old.phone, old.notes);
    INSERT INTO contacts_fts(rowid, name, email, company, phone, notes)
    VALUES (new.rowid, new.name, new.email, new.company, new.phone, new.notes);
END;

----------------------------------------------
-- TAGS
----------------------------------------------
CREATE TABLE tags (
    id    TEXT PRIMARY KEY,               -- ULID
    name  TEXT NOT NULL UNIQUE,
    color TEXT NOT NULL DEFAULT '#6B7280'
);

CREATE TABLE contact_tags (
    contact_id TEXT NOT NULL REFERENCES contacts(id) ON DELETE CASCADE,
    tag_id     TEXT NOT NULL REFERENCES tags(id) ON DELETE CASCADE,
    PRIMARY KEY (contact_id, tag_id)
);

CREATE INDEX idx_contact_tags_contact ON contact_tags(contact_id);
CREATE INDEX idx_contact_tags_tag ON contact_tags(tag_id);

----------------------------------------------
-- USER SHORTCUTS
----------------------------------------------
CREATE TABLE user_shortcuts (
    id      TEXT PRIMARY KEY,
    action  TEXT NOT NULL UNIQUE,
    keys    TEXT NOT NULL,
    context TEXT NOT NULL DEFAULT 'global'
);

----------------------------------------------
-- SETTINGS
----------------------------------------------
CREATE TABLE settings (
    key   TEXT PRIMARY KEY,
    value TEXT NOT NULL                    -- JSON-encoded
);

INSERT INTO settings (key, value) VALUES
    ('theme', '"system"'),
    ('sidebar_collapsed', 'false'),
    ('csv_delimiter', '","');
```

Note: No deals or deal_stages tables. Those are Phase 2 additions.

---

## 4. Tauri Command Architecture

### How It Differs from Electron

Electron uses IPC channels (string-based message passing). Tauri uses `#[tauri::command]` - Rust functions that the frontend calls directly via `invoke()`. Arguments and return values are auto-serialized via Serde. Type safety extends from Rust to TypeScript.

```
Frontend (React)  --invoke()--> Tauri Runtime --> Rust #[tauri::command] --> rusqlite --> SQLite
                  <--Result---                <-- Return value (Serde JSON)
```

### AppState

```rust
// src-tauri/src/state.rs
use rusqlite::Connection;
use std::sync::Mutex;

pub struct AppState {
    pub db: Mutex<Connection>,
}

impl AppState {
    pub fn new(db_path: &str) -> Self {
        let conn = Connection::open(db_path).expect("Failed to open database");
        conn.execute_batch("PRAGMA journal_mode = WAL; PRAGMA foreign_keys = ON;")
            .expect("Failed to set pragmas");
        Self { db: Mutex::new(conn) }
    }
}
```

### Rust Models

```rust
// src-tauri/src/models/contact.rs
use serde::{Deserialize, Serialize};

#[derive(Debug, Serialize, Deserialize, Clone)]
pub struct Contact {
    pub id: String,
    pub name: String,
    pub email: Option<String>,
    pub phone: Option<String>,
    pub company: Option<String>,
    pub title: Option<String>,
    pub notes: Option<String>,
    pub custom_fields: String,          // JSON string
    pub created_at: String,
    pub updated_at: String,
    pub archived_at: Option<String>,
}

#[derive(Debug, Deserialize)]
pub struct CreateContact {
    pub name: String,
    pub email: Option<String>,
    pub phone: Option<String>,
    pub company: Option<String>,
    pub title: Option<String>,
    pub notes: Option<String>,
    pub custom_fields: Option<serde_json::Value>,
}

#[derive(Debug, Deserialize)]
pub struct UpdateContact {
    pub name: Option<String>,
    pub email: Option<String>,
    pub phone: Option<String>,
    pub company: Option<String>,
    pub title: Option<String>,
    pub notes: Option<String>,
    pub custom_fields: Option<serde_json::Value>,
}

#[derive(Debug, Deserialize, Default)]
pub struct ContactFilter {
    pub search: Option<String>,
    pub company: Option<String>,
    pub tag: Option<String>,
    pub archived: Option<bool>,
    pub limit: Option<i64>,
}
```

### Rust Database Layer

```rust
// src-tauri/src/db/contacts.rs
use rusqlite::{params, Connection, Result};
use crate::models::contact::*;

pub fn list_contacts(conn: &Connection, filter: &ContactFilter) -> Result<Vec<Contact>> {
    let mut query = String::from(
        "SELECT id, name, email, phone, company, title, notes, custom_fields, \
         created_at, updated_at, archived_at FROM contacts WHERE 1=1"
    );
    let mut param_values: Vec<Box<dyn rusqlite::types::ToSql>> = Vec::new();

    if filter.archived.unwrap_or(false) {
        query.push_str(" AND archived_at IS NOT NULL");
    } else {
        query.push_str(" AND archived_at IS NULL");
    }

    if let Some(ref company) = filter.company {
        query.push_str(" AND company = ?");
        param_values.push(Box::new(company.clone()));
    }

    if let Some(ref tag) = filter.tag {
        query.push_str(" AND id IN (SELECT contact_id FROM contact_tags WHERE tag_id = ?)");
        param_values.push(Box::new(tag.clone()));
    }

    query.push_str(" ORDER BY created_at DESC");

    if let Some(limit) = filter.limit {
        query.push_str(&format!(" LIMIT {}", limit));
    }

    let mut stmt = conn.prepare(&query)?;
    let params_refs: Vec<&dyn rusqlite::types::ToSql> = param_values.iter().map(|p| p.as_ref()).collect();

    let contacts = stmt.query_map(params_refs.as_slice(), |row| {
        Ok(Contact {
            id: row.get(0)?,
            name: row.get(1)?,
            email: row.get(2)?,
            phone: row.get(3)?,
            company: row.get(4)?,
            title: row.get(5)?,
            notes: row.get(6)?,
            custom_fields: row.get(7)?,
            created_at: row.get(8)?,
            updated_at: row.get(9)?,
            archived_at: row.get(10)?,
        })
    })?.collect::<Result<Vec<_>>>()?;

    Ok(contacts)
}

pub fn create_contact(conn: &Connection, data: &CreateContact) -> Result<Contact> {
    let id = ulid::Ulid::new().to_string();
    let now = chrono::Utc::now().to_rfc3339();
    let custom_fields = serde_json::to_string(
        &data.custom_fields.clone().unwrap_or(serde_json::json!({}))
    ).unwrap_or_default();

    conn.execute(
        "INSERT INTO contacts (id, name, email, phone, company, title, notes, custom_fields, created_at, updated_at) \
         VALUES (?1, ?2, ?3, ?4, ?5, ?6, ?7, ?8, ?9, ?10)",
        params![id, data.name, data.email, data.phone, data.company, data.title, data.notes, custom_fields, now, now],
    )?;

    get_contact(conn, &id)?.ok_or_else(|| rusqlite::Error::QueryReturnedNoRows)
}

pub fn get_contact(conn: &Connection, id: &str) -> Result<Option<Contact>> {
    conn.query_row(
        "SELECT id, name, email, phone, company, title, notes, custom_fields, \
         created_at, updated_at, archived_at FROM contacts WHERE id = ?1",
        params![id],
        |row| Ok(Contact {
            id: row.get(0)?,
            name: row.get(1)?,
            email: row.get(2)?,
            phone: row.get(3)?,
            company: row.get(4)?,
            title: row.get(5)?,
            notes: row.get(6)?,
            custom_fields: row.get(7)?,
            created_at: row.get(8)?,
            updated_at: row.get(9)?,
            archived_at: row.get(10)?,
        }),
    ).optional()
}
```

### Rust Commands Layer

```rust
// src-tauri/src/commands/contacts.rs
use tauri::State;
use crate::state::AppState;
use crate::db;
use crate::models::contact::*;

#[tauri::command]
pub fn list_contacts(
    state: State<'_, AppState>,
    filter: ContactFilter,
) -> Result<Vec<Contact>, String> {
    let conn = state.db.lock().map_err(|e| e.to_string())?;
    db::contacts::list_contacts(&conn, &filter).map_err(|e| e.to_string())
}

#[tauri::command]
pub fn get_contact(
    state: State<'_, AppState>,
    id: String,
) -> Result<Option<Contact>, String> {
    let conn = state.db.lock().map_err(|e| e.to_string())?;
    db::contacts::get_contact(&conn, &id).map_err(|e| e.to_string())
}

#[tauri::command]
pub fn create_contact(
    state: State<'_, AppState>,
    data: CreateContact,
) -> Result<Contact, String> {
    let conn = state.db.lock().map_err(|e| e.to_string())?;
    db::contacts::create_contact(&conn, &data).map_err(|e| e.to_string())
}

#[tauri::command]
pub fn update_contact(
    state: State<'_, AppState>,
    id: String,
    data: UpdateContact,
) -> Result<Contact, String> {
    let conn = state.db.lock().map_err(|e| e.to_string())?;
    db::contacts::update_contact(&conn, &id, &data).map_err(|e| e.to_string())
}

#[tauri::command]
pub fn delete_contact(
    state: State<'_, AppState>,
    id: String,
) -> Result<(), String> {
    let conn = state.db.lock().map_err(|e| e.to_string())?;
    db::contacts::delete_contact(&conn, &id).map_err(|e| e.to_string())
}

#[tauri::command]
pub fn archive_contact(
    state: State<'_, AppState>,
    id: String,
) -> Result<(), String> {
    let conn = state.db.lock().map_err(|e| e.to_string())?;
    db::contacts::archive_contact(&conn, &id).map_err(|e| e.to_string())
}

#[tauri::command]
pub fn search_contacts(
    state: State<'_, AppState>,
    query: String,
    limit: Option<i64>,
) -> Result<Vec<Contact>, String> {
    let conn = state.db.lock().map_err(|e| e.to_string())?;
    db::search::search_contacts(&conn, &query, limit.unwrap_or(50)).map_err(|e| e.to_string())
}

#[tauri::command]
pub fn import_csv(
    state: State<'_, AppState>,
    file_path: String,
    mapping: std::collections::HashMap<String, String>,
) -> Result<ImportResult, String> {
    let conn = state.db.lock().map_err(|e| e.to_string())?;
    db::import_export::import_csv(&conn, &file_path, &mapping).map_err(|e| e.to_string())
}
```

### Command Registration

```rust
// src-tauri/src/lib.rs
use tauri::Manager;

mod commands;
mod db;
mod error;
mod models;
mod state;

pub fn run() {
    let db_path = dirs::data_dir()
        .unwrap()
        .join("com.localcrm.app")
        .join("localcrm.db");

    // Ensure directory exists
    std::fs::create_dir_all(db_path.parent().unwrap()).unwrap();

    let app_state = state::AppState::new(db_path.to_str().unwrap());
    db::schema::initialize(&app_state.db.lock().unwrap()).unwrap();

    tauri::Builder::default()
        .plugin(tauri_plugin_dialog::init())
        .plugin(tauri_plugin_fs::init())
        .plugin(tauri_plugin_updater::Builder::new().build())
        .plugin(tauri_plugin_shell::init())
        .manage(app_state)
        .invoke_handler(tauri::generate_handler![
            commands::contacts::list_contacts,
            commands::contacts::get_contact,
            commands::contacts::create_contact,
            commands::contacts::update_contact,
            commands::contacts::delete_contact,
            commands::contacts::archive_contact,
            commands::contacts::restore_contact,
            commands::search::search_contacts,
            commands::tags::list_tags,
            commands::tags::create_tag,
            commands::tags::delete_tag,
            commands::tags::add_tag_to_contact,
            commands::tags::remove_tag_from_contact,
            commands::tags::get_tags_for_contact,
            commands::import_export::import_csv,
            commands::import_export::export_csv,
            commands::settings::get_setting,
            commands::settings::set_setting,
            commands::shortcuts::list_shortcuts,
            commands::shortcuts::update_shortcut,
            commands::shortcuts::reset_shortcuts,
        ])
        .run(tauri::generate_context!())
        .expect("error while running tauri application");
}
```

### Frontend Typed Invoke Wrapper

```typescript
// src/lib/tauri.ts
import { invoke } from '@tauri-apps/api/core';

export const api = {
    contacts: {
        list: (filter: ContactFilter = {}) =>
            invoke<Contact[]>('list_contacts', { filter }),
        get: (id: string) =>
            invoke<Contact | null>('get_contact', { id }),
        create: (data: CreateContact) =>
            invoke<Contact>('create_contact', { data }),
        update: (id: string, data: UpdateContact) =>
            invoke<Contact>('update_contact', { id, data }),
        delete: (id: string) =>
            invoke<void>('delete_contact', { id }),
        archive: (id: string) =>
            invoke<void>('archive_contact', { id }),
        restore: (id: string) =>
            invoke<void>('restore_contact', { id }),
        search: (query: string, limit?: number) =>
            invoke<Contact[]>('search_contacts', { query, limit }),
        importCSV: (filePath: string, mapping: Record<string, string>) =>
            invoke<ImportResult>('import_csv', { filePath, mapping }),
        exportCSV: (filter?: ContactFilter) =>
            invoke<string>('export_csv', { filter }),
    },
    tags: {
        list: () => invoke<Tag[]>('list_tags'),
        create: (name: string, color: string) =>
            invoke<Tag>('create_tag', { name, color }),
        delete: (id: string) => invoke<void>('delete_tag', { id }),
        addToContact: (contactId: string, tagId: string) =>
            invoke<void>('add_tag_to_contact', { contactId, tagId }),
        removeFromContact: (contactId: string, tagId: string) =>
            invoke<void>('remove_tag_from_contact', { contactId, tagId }),
        getForContact: (contactId: string) =>
            invoke<Tag[]>('get_tags_for_contact', { contactId }),
    },
    settings: {
        get: <T>(key: string) => invoke<T>('get_setting', { key }),
        set: (key: string, value: unknown) =>
            invoke<void>('set_setting', { key, value: JSON.stringify(value) }),
    },
    shortcuts: {
        list: () => invoke<UserShortcut[]>('list_shortcuts'),
        update: (action: string, keys: string) =>
            invoke<UserShortcut>('update_shortcut', { action, keys }),
        reset: () => invoke<UserShortcut[]>('reset_shortcuts'),
    },
};
```

---

## 5. State Management

### Contact Store

```typescript
// src/stores/contactStore.ts
import { create } from 'zustand';
import { api } from '../lib/tauri';

interface ContactStore {
    contacts: Contact[];
    selectedId: string | null;
    filter: ContactFilter;
    loading: boolean;

    loadContacts: () => Promise<void>;
    selectContact: (id: string | null) => void;
    selectNext: () => void;
    selectPrevious: () => void;
    createContact: (data: CreateContact) => Promise<Contact>;
    updateContact: (id: string, data: UpdateContact) => Promise<void>;
    deleteContact: (id: string) => Promise<void>;
    archiveContact: (id: string) => Promise<void>;
    restoreContact: (id: string) => Promise<void>;
    setFilter: (filter: Partial<ContactFilter>) => void;
}

export const useContactStore = create<ContactStore>((set, get) => ({
    contacts: [],
    selectedId: null,
    filter: {},
    loading: false,

    loadContacts: async () => {
        set({ loading: true });
        const contacts = await api.contacts.list(get().filter);
        set({ contacts, loading: false });
    },

    selectContact: (id) => set({ selectedId: id }),

    selectNext: () => {
        const { contacts, selectedId } = get();
        const idx = contacts.findIndex((c) => c.id === selectedId);
        const nextIdx = Math.min(idx + 1, contacts.length - 1);
        set({ selectedId: contacts[nextIdx]?.id ?? null });
    },

    selectPrevious: () => {
        const { contacts, selectedId } = get();
        const idx = contacts.findIndex((c) => c.id === selectedId);
        const prevIdx = Math.max(idx - 1, 0);
        set({ selectedId: contacts[prevIdx]?.id ?? null });
    },

    createContact: async (data) => {
        const contact = await api.contacts.create(data);
        set((s) => ({ contacts: [contact, ...s.contacts] }));
        return contact;
    },

    updateContact: async (id, data) => {
        const updated = await api.contacts.update(id, data);
        set((s) => ({
            contacts: s.contacts.map((c) => (c.id === id ? updated : c)),
        }));
    },

    deleteContact: async (id) => {
        await api.contacts.delete(id);
        set((s) => ({
            contacts: s.contacts.filter((c) => c.id !== id),
            selectedId: s.selectedId === id ? null : s.selectedId,
        }));
    },

    archiveContact: async (id) => {
        await api.contacts.archive(id);
        set((s) => ({
            contacts: s.contacts.filter((c) => c.id !== id),
            selectedId: s.selectedId === id ? null : s.selectedId,
        }));
    },

    restoreContact: async (id) => {
        await api.contacts.restore(id);
        await get().loadContacts();
    },

    setFilter: (filter) => {
        set((s) => ({ filter: { ...s.filter, ...filter } }));
        get().loadContacts();
    },
}));
```

### UI Store

```typescript
// src/stores/uiStore.ts
import { create } from 'zustand';

type View = 'contacts' | 'settings';

interface UIStore {
    view: View;
    sidebarCollapsed: boolean;
    commandPaletteOpen: boolean;
    keyboardHelpOpen: boolean;
    searchFocused: boolean;
    modalStack: string[];

    navigate: (view: View) => void;
    toggleSidebar: () => void;
    toggleCommandPalette: () => void;
    toggleKeyboardHelp: () => void;
    setSearchFocused: (focused: boolean) => void;
    pushModal: (id: string) => void;
    popModal: () => void;
}

export const useUIStore = create<UIStore>((set) => ({
    view: 'contacts',
    sidebarCollapsed: false,
    commandPaletteOpen: false,
    keyboardHelpOpen: false,
    searchFocused: false,
    modalStack: [],

    navigate: (view) => set({ view }),
    toggleSidebar: () => set((s) => ({ sidebarCollapsed: !s.sidebarCollapsed })),
    toggleCommandPalette: () => set((s) => ({ commandPaletteOpen: !s.commandPaletteOpen })),
    toggleKeyboardHelp: () => set((s) => ({ keyboardHelpOpen: !s.keyboardHelpOpen })),
    setSearchFocused: (focused) => set({ searchFocused: focused }),
    pushModal: (id) => set((s) => ({ modalStack: [...s.modalStack, id] })),
    popModal: () => set((s) => ({ modalStack: s.modalStack.slice(0, -1) })),
}));
```

Note: No dealStore. Views are just `contacts` and `settings`.

---

## 6. Keyboard Shortcut System

### Full Shortcut Map

| Key | Action | Context | Customizable |
|-----|--------|---------|-------------|
| `Mod+K` | Open command palette | Global | Yes |
| `Mod+N` | New contact | Global | Yes |
| `Mod+F` | Focus search bar | Global | Yes |
| `Mod+,` | Open settings | Global | Yes |
| `Mod+/` | Toggle keyboard help | Global | Yes |
| `Escape` | Close modal / deselect / blur search | Global | No |
| `g c` | Go to contacts | Global (chord) | Yes |
| `g s` | Go to settings | Global (chord) | Yes |
| `j` | Select next contact | Contacts | Yes |
| `k` | Select previous contact | Contacts | Yes |
| `Enter` | Open selected contact | Contacts | No |
| `x` | Toggle select current | Contacts | Yes |
| `Shift+X` | Select range | Contacts | Yes |
| `e` | Edit selected contact | Contacts | Yes |
| `d d` | Delete (with confirmation) | Contacts (chord) | Yes |
| `t` | Add tag | Contacts | Yes |
| `n` | Add note | Contacts | Yes |
| `a` | Archive selected | Contacts | Yes |

No deal shortcuts (g+d, m, v) - those are Phase 2.

### Keyboard Engine

Same implementation as Plan A (see Plan A section 6). The KeyboardEngine class is identical since it's a frontend-only concern. Chord detection, context routing, input-focus guard all work the same way.

---

## 7. Search Implementation

### FTS5 in Rust

```rust
// src-tauri/src/db/search.rs
use rusqlite::{params, Connection, Result};
use crate::models::contact::Contact;

pub fn search_contacts(
    conn: &Connection,
    query: &str,
    limit: i64,
) -> Result<Vec<Contact>> {
    // Append * for prefix matching: "john" -> "john*"
    let fts_query = if query.ends_with('*') {
        query.to_string()
    } else {
        format!("{}*", query)
    };

    let mut stmt = conn.prepare(
        "SELECT c.id, c.name, c.email, c.phone, c.company, c.title, c.notes, \
         c.custom_fields, c.created_at, c.updated_at, c.archived_at, rank \
         FROM contacts_fts \
         JOIN contacts c ON contacts_fts.rowid = c.rowid \
         WHERE contacts_fts MATCH ?1 \
         AND c.archived_at IS NULL \
         ORDER BY rank \
         LIMIT ?2"
    )?;

    let contacts = stmt.query_map(params![fts_query, limit], |row| {
        Ok(Contact {
            id: row.get(0)?,
            name: row.get(1)?,
            email: row.get(2)?,
            phone: row.get(3)?,
            company: row.get(4)?,
            title: row.get(5)?,
            notes: row.get(6)?,
            custom_fields: row.get(7)?,
            created_at: row.get(8)?,
            updated_at: row.get(9)?,
            archived_at: row.get(10)?,
        })
    })?.collect::<Result<Vec<_>>>()?;

    Ok(contacts)
}
```

### Performance

Rust's rusqlite executes FTS5 queries faster than Electron's better-sqlite3 (no IPC overhead, no JS-to-native bridge). Expected:
- 100K contacts: <30ms (vs <50ms in Electron)
- 500K contacts: <150ms (vs <200ms in Electron)
- The invoke() overhead is ~1-2ms (vs ~5-10ms for Electron IPC)

---

## 8. Custom Fields

### Rust Struct

```rust
// src-tauri/src/models/custom_field.rs
use serde::{Deserialize, Serialize};

#[derive(Debug, Serialize, Deserialize, Clone)]
#[serde(rename_all = "camelCase")]
pub struct CustomFieldDefinition {
    pub id: String,
    pub name: String,
    pub key: String,
    pub field_type: CustomFieldType,
    #[serde(skip_serializing_if = "Option::is_none")]
    pub options: Option<Vec<String>>,
    #[serde(default)]
    pub required: bool,
    #[serde(skip_serializing_if = "Option::is_none")]
    pub default_value: Option<serde_json::Value>,
    pub position: i32,
}

#[derive(Debug, Serialize, Deserialize, Clone)]
#[serde(rename_all = "lowercase")]
pub enum CustomFieldType {
    Text,
    Number,
    Date,
    Select,
    #[serde(rename = "multi-select")]
    MultiSelect,
    Url,
    Email,
    Phone,
}
```

### Storage

Custom field definitions stored in settings table:
```sql
INSERT INTO settings (key, value) VALUES ('custom_field_definitions', '[...]');
```

Contact custom field values stored as JSON in contacts.custom_fields:
```json
{"linkedin_url": "https://linkedin.com/in/johndoe", "source": "referral", "priority": 3}
```

Serde handles all serialization/deserialization automatically. The `custom_fields` column is a `String` in Rust, parsed to `serde_json::Value` when needed for filtering or display.

---

## 9. CSV Import/Export

### Rust CSV Import (recommended over papaparse)

```rust
// src-tauri/src/db/import_export.rs
use csv::Reader;
use rusqlite::{params, Connection, Result};
use std::collections::HashMap;
use crate::models::contact::ImportResult;

pub fn import_csv(
    conn: &Connection,
    file_path: &str,
    mapping: &HashMap<String, String>,  // csv_column -> contact_field
) -> Result<ImportResult> {
    let mut result = ImportResult {
        imported: 0,
        skipped: 0,
        errors: Vec::new(),
    };

    let mut rdr = Reader::from_path(file_path)
        .map_err(|e| rusqlite::Error::InvalidParameterName(e.to_string()))?;

    let headers = rdr.headers()
        .map_err(|e| rusqlite::Error::InvalidParameterName(e.to_string()))?
        .clone();

    // Batch insert in transaction
    let tx = conn.unchecked_transaction()?;

    let check_duplicate = tx.prepare_cached(
        "SELECT id FROM contacts WHERE email = ?1"
    )?;

    let insert = tx.prepare_cached(
        "INSERT INTO contacts (id, name, email, phone, company, title, notes, custom_fields, created_at, updated_at) \
         VALUES (?1, ?2, ?3, ?4, ?5, ?6, ?7, ?8, ?9, ?10)"
    )?;

    let mut batch_count = 0;

    for (row_idx, record) in rdr.records().enumerate() {
        let record = match record {
            Ok(r) => r,
            Err(e) => {
                result.errors.push(format!("Row {}: {}", row_idx + 1, e));
                continue;
            }
        };

        let mut contact_data: HashMap<String, String> = HashMap::new();
        let mut custom_fields: HashMap<String, String> = HashMap::new();

        for (csv_col, contact_field) in mapping.iter() {
            let col_idx = headers.iter().position(|h| h == csv_col.as_str());
            if let Some(idx) = col_idx {
                let value = record.get(idx).unwrap_or("").to_string();
                if !value.is_empty() {
                    match contact_field.as_str() {
                        "name" | "email" | "phone" | "company" | "title" | "notes" => {
                            contact_data.insert(contact_field.clone(), value);
                        }
                        _ => {
                            custom_fields.insert(contact_field.clone(), value);
                        }
                    }
                }
            }
        }

        // Skip if no name
        let name = match contact_data.get("name") {
            Some(n) if !n.is_empty() => n.clone(),
            _ => {
                result.errors.push(format!("Row {}: missing name", row_idx + 1));
                continue;
            }
        };

        // Duplicate check
        let email = contact_data.get("email").cloned();
        if let Some(ref e) = email {
            if check_duplicate.exists(params![e]).unwrap_or(false) {
                result.skipped += 1;
                continue;
            }
        }

        let id = ulid::Ulid::new().to_string();
        let now = chrono::Utc::now().to_rfc3339();
        let cf_json = serde_json::to_string(&custom_fields).unwrap_or_default();

        insert.execute(params![
            id,
            name,
            email,
            contact_data.get("phone"),
            contact_data.get("company"),
            contact_data.get("title"),
            contact_data.get("notes"),
            cf_json,
            now,
            now,
        ])?;

        result.imported += 1;
        batch_count += 1;

        // Commit every 500 rows for progress
        if batch_count >= 500 {
            batch_count = 0;
        }
    }

    tx.commit()?;
    Ok(result)
}
```

### Why Rust CSV over papaparse

- Rust `csv` crate processes 1M rows in ~1.5s vs papaparse ~5s
- No JSON serialization overhead for large datasets
- Memory-efficient streaming (no full file in memory)
- Transaction batching is simpler in Rust (direct connection access)
- For 10K contacts: ~1s in Rust vs ~3s in papaparse

### Frontend Field Mapper

The CSVImport and FieldMapper components are identical to Plan A. The frontend reads the first 100 rows via a Tauri command for preview, then sends the full file path + mapping to the Rust import command. The Rust side handles all parsing and insertion.

---

## 10. Component Architecture

### ContactList

```typescript
// Key behaviors:
// - Virtual scrolling via @tanstack/react-virtual
// - Fixed row height: 64px
// - j/k keyboard navigation via contactStore.selectNext/Previous
// - Multi-select with x/Shift+X
// - 5 rows overscan
// - Scrolls to selected item on keyboard navigation

// Performance:
// - 100K contacts at 60fps
// - Only renders ~15-20 visible rows
// - Row memoization with React.memo
```

### ContactDetail

```typescript
// Key behaviors:
// - Read-only view by default
// - Toggle edit mode with 'e' key
// - Custom fields rendered via CustomFieldRenderer
// - Tags as colored chips with click-to-remove
// - Notes section with auto-save (debounced 500ms)
// - Delete with 'd d' chord (confirmation dialog)
// - Archive with 'a' key

// No deals section (Phase 2)
```

### CommandPalette

```typescript
// Actions (no deal actions):
const actions: CommandAction[] = [
    { id: 'nav.contacts', label: 'Go to Contacts', shortcut: 'g c', handler: () => navigate('contacts') },
    { id: 'nav.settings', label: 'Go to Settings', shortcut: 'g s', handler: () => navigate('settings') },
    { id: 'contact.create', label: 'New Contact', shortcut: 'Mod+N', handler: () => openCreateContact() },
    { id: 'import', label: 'Import Contacts (CSV)', handler: () => openImport() },
    { id: 'export', label: 'Export Contacts (CSV)', handler: () => exportContacts() },
    { id: 'search', label: 'Search Contacts', shortcut: 'Mod+F', handler: () => focusSearch() },
    { id: 'keyboard', label: 'Keyboard Shortcuts', shortcut: 'Mod+/', handler: () => toggleKeyboardHelp() },
];
```

### SearchBar, Sidebar, TagInput, CustomFieldRenderer

Same implementation as Plan A. The frontend React components are identical between Electron and Tauri. Only the backend communication layer differs (`window.api.contacts.search()` vs `invoke('search_contacts')`).

---

## 11. Build and Distribution

### tauri.conf.json

```json
{
  "$schema": "https://raw.githubusercontent.com/nicklasbrandt/tauri-v2-schema/main/tauri-conf-v2.schema.json",
  "productName": "LocalCRM",
  "version": "0.1.0",
  "identifier": "com.localcrm.app",
  "build": {
    "frontendDist": "../dist",
    "devUrl": "http://localhost:5173",
    "beforeBuildCommand": "npm run build",
    "beforeDevCommand": "npm run dev"
  },
  "app": {
    "windows": [
      {
        "title": "LocalCRM",
        "width": 1200,
        "height": 800,
        "minWidth": 900,
        "minHeight": 600,
        "decorations": true,
        "resizable": true
      }
    ],
    "security": {
      "csp": "default-src 'self'; style-src 'self' 'unsafe-inline'"
    }
  },
  "bundle": {
    "active": true,
    "targets": ["dmg", "nsis", "deb", "appimage"],
    "icon": [
      "icons/32x32.png",
      "icons/128x128.png",
      "icons/128x128@2x.png",
      "icons/icon.icns",
      "icons/icon.ico"
    ],
    "macOS": {
      "signingIdentity": null,
      "minimumSystemVersion": "10.15"
    },
    "windows": {
      "certificateThumbprint": null,
      "digestAlgorithm": "sha256"
    }
  },
  "plugins": {
    "updater": {
      "endpoints": [
        "https://github.com/YOUR_ORG/local-crm/releases/latest/download/latest.json"
      ],
      "pubkey": "YOUR_PUBLIC_KEY"
    }
  }
}
```

### Bundle Size Comparison

| Platform | Tauri | Electron |
|----------|-------|----------|
| Mac (.dmg) | ~8-12 MB | ~165 MB |
| Windows (.msi) | ~6-10 MB | ~120 MB |
| Linux (.AppImage) | ~8-12 MB | ~180 MB |

### GitHub Actions with tauri-action

```yaml
name: Build and Release

on:
  push:
    tags:
      - 'v*'

jobs:
  build:
    strategy:
      matrix:
        include:
          - platform: macos-latest
            args: '--target universal-apple-darwin'
          - platform: ubuntu-22.04
            args: ''
          - platform: windows-latest
            args: ''
    runs-on: ${{ matrix.os || matrix.platform }}
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 22
      - uses: dtolnay/rust-toolchain@stable
        with:
          targets: ${{ matrix.platform == 'macos-latest' && 'aarch64-apple-darwin,x86_64-apple-darwin' || '' }}
      - name: Install Linux dependencies
        if: matrix.platform == 'ubuntu-22.04'
        run: |
          sudo apt-get update
          sudo apt-get install -y libwebkit2gtk-4.1-dev libappindicator3-dev librsvg2-dev patchelf
      - run: npm ci
      - uses: tauri-apps/tauri-action@v0
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          APPLE_CERTIFICATE: ${{ secrets.APPLE_CERTIFICATE }}
          APPLE_CERTIFICATE_PASSWORD: ${{ secrets.APPLE_CERTIFICATE_PASSWORD }}
          APPLE_SIGNING_IDENTITY: ${{ secrets.APPLE_SIGNING_IDENTITY }}
          APPLE_ID: ${{ secrets.APPLE_ID }}
          APPLE_PASSWORD: ${{ secrets.APPLE_PASSWORD }}
          APPLE_TEAM_ID: ${{ secrets.APPLE_TEAM_ID }}
          TAURI_SIGNING_PRIVATE_KEY: ${{ secrets.TAURI_SIGNING_PRIVATE_KEY }}
        with:
          tagName: v__VERSION__
          releaseName: 'LocalCRM v__VERSION__'
          releaseBody: 'See the changelog for details.'
          releaseDraft: true
          prerelease: false
          args: ${{ matrix.args }}
```

---

## 12. Testing Strategy

### Rust Backend Tests

```rust
// src-tauri/src/db/contacts.rs
#[cfg(test)]
mod tests {
    use super::*;
    use rusqlite::Connection;

    fn setup_test_db() -> Connection {
        let conn = Connection::open_in_memory().unwrap();
        crate::db::schema::initialize(&conn).unwrap();
        conn
    }

    #[test]
    fn test_create_and_get_contact() {
        let conn = setup_test_db();
        let data = CreateContact {
            name: "John Doe".into(),
            email: Some("john@example.com".into()),
            phone: None,
            company: Some("Acme".into()),
            title: None,
            notes: None,
            custom_fields: None,
        };

        let contact = create_contact(&conn, &data).unwrap();
        assert_eq!(contact.name, "John Doe");
        assert_eq!(contact.email, Some("john@example.com".into()));

        let fetched = get_contact(&conn, &contact.id).unwrap().unwrap();
        assert_eq!(fetched.id, contact.id);
    }

    #[test]
    fn test_search_contacts() {
        let conn = setup_test_db();
        // Create test contacts
        create_contact(&conn, &CreateContact {
            name: "Alice Johnson".into(),
            email: Some("alice@test.com".into()),
            ..Default::default()
        }).unwrap();

        let results = crate::db::search::search_contacts(&conn, "alice", 50).unwrap();
        assert_eq!(results.len(), 1);
        assert_eq!(results[0].name, "Alice Johnson");
    }

    #[test]
    fn test_import_csv() {
        // Write temp CSV, import, verify contacts created
    }
}
```

### Frontend Tests

Same vitest + @testing-library/react setup as Plan A. Mock the `invoke()` function:

```typescript
// tests/setup.ts
import { vi } from 'vitest';

vi.mock('@tauri-apps/api/core', () => ({
    invoke: vi.fn(),
}));
```

### E2E Tests

Tauri supports WebDriver protocol. Playwright can connect via WebDriver:

```typescript
// tests/e2e/contacts.spec.ts
// Note: Tauri E2E testing is less mature than Electron.
// Use @tauri-apps/api/test or WebDriver approach.
```

### Coverage Targets

| Area | Target |
|------|--------|
| Rust backend (db, commands) | 80%+ |
| Zustand stores | 80%+ |
| Frontend utilities | 90%+ |
| React components | 60%+ |
| Overall (frontend) | 65%+ |
| Overall (Rust) | 75%+ |

---

## 13. Performance Targets

| Metric | Target | How to Measure |
|--------|--------|---------------|
| Cold start (app ready) | <1.8s | Time from launch to first render (Tauri window ready) |
| Search (100K contacts) | <30ms | Rust FTS5 query time (no IPC overhead) |
| List scroll (10K contacts) | 60fps | DevTools Performance tab |
| Contact create | <10ms | invoke() round-trip |
| Contact update | <8ms | invoke() round-trip |
| CSV import (10K rows) | <2s | Rust csv crate + batch transaction |
| CSV import (100K rows) | <15s | Rust csv crate + batch transaction |
| Memory (idle, empty DB) | <50MB | OS Activity Monitor |
| Memory (100K contacts) | <150MB | OS Activity Monitor |
| FTS5 index rebuild (100K) | <2s | Rust timing |
| Bundle size (installer) | ~8-12MB | File size |
| Database file (100K contacts) | ~50MB | File size |

---

## 14. Rust Learning Notes

### What You Need to Know

The Tauri backend for this CRM is thin: ~600-800 lines of Rust total. You don't need deep Rust expertise.

**Essential concepts (cover these first):**

| Concept | Why You Need It | Where It Shows Up |
|---------|----------------|-------------------|
| Variables and ownership | Rust's core differentiator | Every function |
| `String` vs `&str` | Most common confusion | Function parameters |
| `Option<T>` | Nullable fields | Contact email, phone, etc. |
| `Result<T, E>` | Error handling | Every database operation |
| Structs + `impl` | Data models | Contact, Tag, etc. |
| `#[derive(...)]` | Auto-generate traits | Serialize, Deserialize on all models |
| `Mutex<T>` | Thread-safe DB access | AppState.db |
| Pattern matching (`match`, `if let`) | Handling Option/Result | Throughout |
| `?` operator | Error propagation | Every function that returns Result |

**What you can mostly ignore:**

| Concept | Why You Can Skip It |
|---------|-------------------|
| Lifetimes (`'a`) | Compiler infers them for this codebase |
| Traits (advanced) | Only use derived traits (Serialize, Deserialize, Debug) |
| Generics (advanced) | Not needed for CRUD |
| Macros (writing them) | Tauri's macros handle everything |
| Async Rust (deep) | Tauri handles the async runtime, your commands can be sync |
| Unsafe | Never needed for this project |

**Resources:**

1. The Rust Book, chapters 1-10 (skip 11+): https://doc.rust-lang.org/book/
2. Tauri v2 guides: https://v2.tauri.app/start/
3. rusqlite docs: https://docs.rs/rusqlite/latest/

**The key insight:** 90% of your time is spent in the React frontend, which is identical between Electron and Tauri. The Rust backend is essentially a typed SQLite wrapper. If you can write SQL and define structs, you can write this backend.

---

## 15. Day-by-Day Implementation Schedule

Day 0 (parallel to everything): Launch landing page + LinkedIn/Reddit ads ($2,200). Runs throughout build.

### Week 1: Foundation + Contacts

| Day | Instance 1 | Instance 2 | Instance 3 | Integration |
|-----|-----------|-----------|-----------|-------------|
| 1 | Tauri + Vite + React + TypeScript project init. Cargo init. Install all Rust dependencies (Cargo.toml). Verify `cargo tauri dev` launches blank window. | Tailwind 4 setup. Base CSS variables, dark theme. App.tsx shell. Sidebar skeleton (contacts, settings nav). TopBar skeleton. | Rust project structure: create db/, commands/, models/ modules. state.rs with AppState + Mutex<Connection>. schema.rs with full SQL schema. Run schema on app start. | Merge. App launches with dark theme shell, empty sidebar, SQLite database created with all tables. |
| 2 | Rust models: Contact, CreateContact, UpdateContact, ContactFilter structs with Serde derives. db/contacts.rs: list, get, create, update, delete functions. | ContactList component with @tanstack/react-virtual. ContactRow component (64px height). Hard-coded mock data to verify scrolling works. | commands/contacts.rs: all #[tauri::command] wrappers. Register in lib.rs invoke_handler. Verify invoke works from frontend console. | Merge. Wire ContactList to real Tauri commands. Create contact from console, verify it appears in list. |
| 3 | ContactDetail component (read-only view). ContactForm component (create/edit mode). Zod validation schemas. | db/search.rs: FTS5 search function with prefix matching + BM25. commands/search.rs: search_contacts command. | Zustand stores: contactStore (full implementation), uiStore (view, sidebar, modals). src/lib/tauri.ts typed invoke wrapper. | Merge. Full create/read/update/delete flow working. Type in search bar, see results from FTS5. |
| 4 | Tags: db/tags.rs (CRUD + junction queries). commands/tags.rs. Rust Tag model. | TagInput component: autocomplete dropdown, colored chips, create inline. Wire to tag commands. | Custom fields: Rust CustomFieldDefinition model. Settings storage for field definitions. CustomFieldRenderer (all 8 types). | Merge. Tags and custom fields working. Create contact with tags and custom fields, verify persistence. |
| 5 | Archive/restore in db/contacts.rs. ContactFilter support for archived, company, tag params. | Sidebar: active view highlighting, collapse/expand, contact count. Navigation between contacts and settings views. | Keyboard engine (keyboard.ts): chord detection, context routing, isInputFocused. Register Mod+K, Mod+N, Escape, j/k. | Full integration. Generate 1000 test contacts. Verify scroll performance, search speed, keyboard shortcuts. Fix issues. |

### Week 2: Features Sprint (Full Parallel)

| Day | Instance 1 | Instance 2 | Instance 3 | Integration |
|-----|-----------|-----------|-----------|-------------|
| 6 | Custom field definitions management UI. Add/edit/delete/reorder definitions. Save to settings table via Tauri command. | CSV import: Rust csv crate implementation (db/import_export.rs). Batch transactions of 500. Duplicate detection on email. import_csv command. | Command palette (cmdk): action registry, fuzzy search, styled with dark theme. All available actions registered. | - |
| 7 | Custom field filtering: modify list_contacts to support json_extract queries. Sort by custom field value. UI filter controls. | CSVImport component: file picker (tauri-plugin-dialog), preview first 10 rows, FieldMapper dropdown UI. Progress indication. | Global shortcuts: Mod+F (search focus), Mod+N (new contact), Mod+, (settings). j/k list navigation with auto-scroll. | - |
| 8 | CSV export: Rust export function. Flatten custom fields into columns. save dialog (tauri-plugin-dialog). export_csv command. | Import error handling: per-row validation errors, skip/retry, final summary. Duplicate strategies (skip, update, import anyway). | Chord shortcuts: g+c (contacts), g+s (settings), d+d (delete with confirm). Chord visual indicator (shows pending key). | - |
| 9 | Shortcut persistence: db/shortcuts.rs CRUD. commands/shortcuts.rs. Default shortcuts seeded on first run. | SQLite backup command: copy database file to user-chosen location. Restore from backup file. Simple backup/restore UI. | Keyboard help modal: grouped shortcut table, toggle with Mod+/. Focus management (react-focus-lock) on all modals. | - |
| 10 | - | - | - | Full merge. End-to-end testing: create contacts, search, import 5K CSV, export, keyboard flow, custom fields, tags. Fix all integration issues. Performance check. |

### Week 3: Polish + Distribution

| Day | Instance 1 | Instance 2 | Instance 3 | Integration |
|-----|-----------|-----------|-----------|-------------|
| 11 | Settings page: theme selector (dark/light/system), CSV delimiter, sidebar default. Rust settings commands. | Contact bulk operations: multi-select with x/Shift+X, bulk archive, bulk delete, bulk tag. | Error handling: Rust thiserror types. Frontend error boundaries. Toast notifications for errors. Graceful degradation. | - |
| 12 | Performance optimization: Rust query analysis, prepared statement caching, connection tuning. Test with 100K contacts. | UI polish: transitions, hover states, loading skeletons, empty states, confirmation dialogs. | Tauri config: bundle identifiers, window size defaults, CSP, capability permissions. Icon generation (all sizes). | - |
| 13 | Mac build + code signing ($99 Apple Developer). DMG universal binary. Test on Intel + Apple Silicon. Notarization. | Windows build. MSI/NSIS installer. Test on Windows 10 + 11. Handle SmartScreen. | Linux build: .deb + AppImage. Test on Ubuntu. Auto-updater: tauri-plugin-updater config, signing keys, test update flow. | All platform builds ready and signed. |
| 14 | GitHub Actions CI/CD: tauri-action matrix build (Mac universal, Windows, Linux). Auto-publish to GitHub Releases. | Performance test suite: 100K contact generation script, search benchmarks, scroll FPS measurement, cold start timing. | E2E tests: critical user flows (create contact, search, import CSV, keyboard shortcuts, settings). Playwright + WebDriver. | CI green. All tests pass. All builds succeed. |
| 15 | Alpha release (v0.1.0). GitHub Release with all binaries. Release notes. | Send download links to landing page signups. Prepare demo script. Record 2-min demo video for outreach. | Bug fixes from any last-minute issues. Final performance check. Announce on relevant communities (HN, Reddit, Twitter). | Alpha shipped. Landing page updated with download links. |

---

## 16. Phase 2 Preview

### Week 4-5: Deals Pipeline
- Add deals + deal_stages tables (schema already designed in Plan A)
- Rust CRUD for deals and stages
- DealBoard kanban component with @dnd-kit/core
- Deal cards, columns, drag-and-drop
- Contact-deal linking
- Additional shortcuts: g+d, m, v
- This is faster in Phase 2 because the Rust backend patterns are established

### Week 6-7: Cloud Sync (PowerSync)
- Postgres backend (Supabase or Railway)
- PowerSync SDK integration
- Sync rules for per-user data isolation
- Conflict resolution: last-write-wins
- Sync status indicator in UI
- Multi-device support

### Week 8-9: Email Integration
- IMAP/SMTP via Rust (lettre crate)
- Conversation threads linked to contacts
- Inbox view
- Email templates

### Ongoing
- Analytics and reporting
- API for integrations
- Mobile companion evaluation
- Team features for agency tier ($150/month)
