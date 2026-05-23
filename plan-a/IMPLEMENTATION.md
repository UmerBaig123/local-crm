# Plan A: Conservative Path - Technical Implementation Plan

**Electron + Contacts + Deals | 4 weeks | 3x Claude Code instances | $50/month**

---

## 1. Technology Stack

### Core Framework

| Package | Version | Purpose | Category |
|---------|---------|---------|----------|
| electron | ^33.0 | Desktop shell, Chromium + Node.js runtime | Framework |
| electron-builder | ^25.0 | Build/package for Mac, Windows, Linux | Build |
| electron-updater | ^6.3 | Auto-update via GitHub Releases | Distribution |

### Frontend

| Package | Version | Purpose | Category |
|---------|---------|---------|----------|
| react | ^19.0 | UI library | UI |
| react-dom | ^19.0 | React DOM renderer | UI |
| typescript | ^5.6 | Type safety | Language |
| vite | ^6.0 | Frontend bundler + HMR | Build |
| @vitejs/plugin-react | ^4.3 | Vite React integration | Build |
| tailwindcss | ^4.0 | Utility-first CSS | Styling |
| zustand | ^5.0 | Lightweight state management | State |
| zod | ^3.24 | Schema validation | Validation |
| cmdk | ^1.0 | Command palette (used by Linear, Raycast) | UI |
| @tanstack/react-virtual | ^3.11 | Virtual scrolling for large lists | UI |
| react-focus-lock | ^2.13 | Focus trapping for modals and accessibility | UI |
| @dnd-kit/core | ^6.3 | Drag-and-drop for deals kanban | UI |
| @dnd-kit/sortable | ^10.0 | Sortable lists within kanban columns | UI |
| lucide-react | ^0.468 | Icon library | UI |
| date-fns | ^4.1 | Date formatting and manipulation | Utility |
| papaparse | ^5.5 | CSV parsing with streaming support | Utility |

### Backend (Main Process)

| Package | Version | Purpose | Category |
|---------|---------|---------|----------|
| better-sqlite3 | ^11.7 | Synchronous SQLite3 for Node.js (with FTS5) | Database |
| ulid | ^2.3 | Sortable unique ID generation | Utility |

### Testing

| Package | Version | Purpose | Category |
|---------|---------|---------|----------|
| vitest | ^2.1 | Unit and component testing | Testing |
| @testing-library/react | ^16.1 | React component testing utilities | Testing |
| playwright | ^1.49 | E2E testing (supports Electron) | Testing |

### Developer Tooling

| Package | Version | Purpose | Category |
|---------|---------|---------|----------|
| eslint | ^9.16 | Linting | Tooling |
| prettier | ^3.4 | Code formatting | Tooling |
| @types/better-sqlite3 | ^7.6 | TypeScript types for better-sqlite3 | Tooling |

---

## 2. Project Structure

```
local-crm/
├── electron/
│   ├── main.ts                  # Electron main process entry
│   ├── preload.ts               # Context bridge for renderer
│   └── ipc/
│       ├── contacts.ts          # Contact CRUD IPC handlers
│       ├── deals.ts             # Deal CRUD + stage management
│       ├── search.ts            # FTS5 search handler
│       ├── tags.ts              # Tag CRUD + contact-tag linking
│       ├── import-export.ts     # CSV import/export handlers
│       ├── settings.ts          # Settings get/set handlers
│       ├── shortcuts.ts         # User shortcut CRUD
│       └── database.ts          # SQLite init, migrations, connection
├── src/
│   ├── App.tsx                  # Root component with routing
│   ├── main.tsx                 # React entry point
│   ├── index.css                # Tailwind imports + global styles
│   ├── components/
│   │   ├── layout/
│   │   │   ├── Sidebar.tsx      # Navigation sidebar (contacts, deals, settings)
│   │   │   ├── TopBar.tsx       # Search bar + action buttons
│   │   │   └── KeyboardHelp.tsx # Shortcut reference modal
│   │   ├── contacts/
│   │   │   ├── ContactList.tsx  # Virtual scrolling contact list
│   │   │   ├── ContactDetail.tsx # Read-only contact view with actions
│   │   │   ├── ContactForm.tsx  # Create/edit form with custom fields
│   │   │   └── ContactRow.tsx   # Single row in virtual list
│   │   ├── deals/
│   │   │   ├── DealBoard.tsx    # Kanban board container
│   │   │   ├── DealColumn.tsx   # Single stage column with value summary
│   │   │   ├── DealCard.tsx     # Draggable deal card
│   │   │   └── DealForm.tsx     # Create/edit deal form
│   │   ├── search/
│   │   │   ├── SearchBar.tsx    # Global search input with hotkey
│   │   │   └── SearchResults.tsx # FTS5 search result list
│   │   ├── shared/
│   │   │   ├── CommandPalette.tsx # cmdk-based command palette
│   │   │   ├── TagInput.tsx      # Tag selector with autocomplete
│   │   │   ├── CustomFieldRenderer.tsx # Renders custom field by type
│   │   │   └── VirtualList.tsx   # Reusable virtual scrolling wrapper
│   │   └── import/
│   │       ├── CSVImport.tsx    # Import wizard with progress
│   │       └── FieldMapper.tsx  # Column-to-field mapping UI
│   ├── hooks/
│   │   ├── useKeyboard.ts      # Shortcut registration + chord detection
│   │   ├── useContacts.ts      # Contact data fetching + mutations
│   │   ├── useDeals.ts         # Deal data fetching + mutations
│   │   ├── useSearch.ts        # Search with debounce
│   │   └── useCommandPalette.ts # Command palette actions
│   ├── stores/
│   │   ├── contactStore.ts     # Contact list, selection, filters
│   │   ├── dealStore.ts        # Deals, stages, board state
│   │   ├── uiStore.ts          # View state, modals, sidebar
│   │   └── shortcutStore.ts    # Custom shortcuts, defaults
│   ├── lib/
│   │   ├── ipc.ts              # Typed IPC wrapper (renderer side)
│   │   ├── keyboard.ts         # Shortcut registry + chord engine
│   │   ├── validators.ts       # Zod schemas for contacts, deals
│   │   └── constants.ts        # Default stages, field types, etc.
│   └── types/
│       ├── contact.ts          # Contact, CreateContact, ContactFilter
│       ├── deal.ts             # Deal, CreateDeal, DealStage, DealFilter
│       └── custom-field.ts     # CustomFieldDefinition, CustomFieldType
├── tests/
│   ├── unit/                   # Store + utility tests
│   ├── component/              # React component tests
│   └── e2e/                    # Playwright E2E tests
├── resources/
│   ├── icon.png                # App icon (1024x1024)
│   ├── icon.icns               # Mac icon
│   └── icon.ico                # Windows icon
├── package.json
├── electron-builder.yml        # Build + distribution config
├── tsconfig.json               # TypeScript config (strict)
├── tsconfig.electron.json      # TypeScript config for main process
├── vite.config.ts              # Vite config for renderer
└── tailwind.config.ts
```

**Directory roles:**

- **electron/** - Main process code. Runs in Node.js. Handles SQLite operations, file system access, IPC handlers. This is the "backend" of the app.
- **electron/ipc/** - One file per domain (contacts, deals, tags). Each file registers `ipcMain.handle()` listeners that the renderer calls.
- **src/** - Renderer process. Standard React app bundled by Vite. Runs in Chromium.
- **src/components/** - Organized by feature (contacts, deals, search, import) plus shared components and layout.
- **src/hooks/** - React hooks that wrap IPC calls and state management. Components call hooks, hooks call IPC, IPC calls SQLite.
- **src/stores/** - Zustand stores. Global state for contacts, deals, UI, shortcuts. Stores call IPC methods to persist.
- **src/lib/** - Utilities. Typed IPC wrapper, keyboard shortcut engine, zod validators, app constants.
- **src/types/** - TypeScript type definitions. Shared between renderer code.
- **tests/** - Separated by test type. Unit tests for pure logic, component tests for React, E2E for full app flows.

---

## 3. Database Schema

```sql
-- Enable WAL mode for concurrent reads during writes
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
    archived_at TEXT                       -- NULL = active, set = archived
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

-- Keep FTS in sync with contacts table
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
    color TEXT NOT NULL DEFAULT '#6B7280' -- Hex color
);

CREATE TABLE contact_tags (
    contact_id TEXT NOT NULL REFERENCES contacts(id) ON DELETE CASCADE,
    tag_id     TEXT NOT NULL REFERENCES tags(id) ON DELETE CASCADE,
    PRIMARY KEY (contact_id, tag_id)
);

CREATE INDEX idx_contact_tags_contact ON contact_tags(contact_id);
CREATE INDEX idx_contact_tags_tag ON contact_tags(tag_id);

----------------------------------------------
-- DEALS
----------------------------------------------
CREATE TABLE deal_stages (
    id       TEXT PRIMARY KEY,            -- ULID
    name     TEXT NOT NULL,
    position INTEGER NOT NULL,            -- Sort order
    color    TEXT NOT NULL DEFAULT '#6B7280'
);

-- Default stages
INSERT INTO deal_stages (id, name, position, color) VALUES
    ('01STAGE_LEAD', 'Lead', 0, '#6B7280'),
    ('01STAGE_QUAL', 'Qualified', 1, '#3B82F6'),
    ('01STAGE_PROP', 'Proposal', 2, '#8B5CF6'),
    ('01STAGE_NEGO', 'Negotiation', 3, '#F59E0B'),
    ('01STAGE_WON',  'Closed Won', 4, '#10B981'),
    ('01STAGE_LOST', 'Closed Lost', 5, '#EF4444');

CREATE TABLE deals (
    id            TEXT PRIMARY KEY,       -- ULID
    title         TEXT NOT NULL,
    value         REAL,                   -- Deal monetary value
    currency      TEXT DEFAULT 'USD',
    stage_id      TEXT NOT NULL REFERENCES deal_stages(id),
    contact_id    TEXT REFERENCES contacts(id) ON DELETE SET NULL,
    notes         TEXT,
    custom_fields TEXT DEFAULT '{}',      -- JSON object
    created_at    TEXT NOT NULL DEFAULT (datetime('now')),
    updated_at    TEXT NOT NULL DEFAULT (datetime('now')),
    closed_at     TEXT                    -- Set when moved to Won/Lost
);

CREATE INDEX idx_deals_stage ON deals(stage_id);
CREATE INDEX idx_deals_contact ON deals(contact_id);
CREATE INDEX idx_deals_value ON deals(value);
CREATE INDEX idx_deals_created ON deals(created_at);

----------------------------------------------
-- USER SHORTCUTS
----------------------------------------------
CREATE TABLE user_shortcuts (
    id      TEXT PRIMARY KEY,
    action  TEXT NOT NULL UNIQUE,          -- e.g. 'navigate.contacts'
    keys    TEXT NOT NULL,                 -- e.g. 'g c' or 'mod+k'
    context TEXT NOT NULL DEFAULT 'global' -- global, contacts, deals
);

----------------------------------------------
-- SETTINGS
----------------------------------------------
CREATE TABLE settings (
    key   TEXT PRIMARY KEY,
    value TEXT NOT NULL                    -- JSON-encoded value
);

-- Default settings
INSERT INTO settings (key, value) VALUES
    ('theme', '"system"'),
    ('sidebar_collapsed', 'false'),
    ('default_currency', '"USD"'),
    ('csv_delimiter', '","');
```

---

## 4. IPC Architecture

### Pattern

```
Renderer (React)  -->  Preload (contextBridge)  -->  Main (Node.js + SQLite)
    invoke()               ipcRenderer                  ipcMain.handle()
    typed API              safe exposure                 better-sqlite3
```

The renderer never accesses Node.js or SQLite directly. All communication goes through typed IPC channels exposed via the preload script.

### TypeScript API Interface

```typescript
// types/electron-api.ts

export interface ElectronAPI {
    contacts: {
        list(filter: ContactFilter): Promise<Contact[]>;
        get(id: string): Promise<Contact | null>;
        create(data: CreateContact): Promise<Contact>;
        update(id: string, data: UpdateContact): Promise<Contact>;
        delete(id: string): Promise<void>;
        archive(id: string): Promise<void>;
        restore(id: string): Promise<void>;
        search(query: string, limit?: number): Promise<Contact[]>;
        importCSV(filePath: string, mapping: FieldMapping): Promise<ImportResult>;
        exportCSV(filter?: ContactFilter): Promise<string>;
    };
    deals: {
        list(filter: DealFilter): Promise<Deal[]>;
        get(id: string): Promise<Deal | null>;
        create(data: CreateDeal): Promise<Deal>;
        update(id: string, data: UpdateDeal): Promise<Deal>;
        delete(id: string): Promise<void>;
        moveStage(id: string, stageId: string): Promise<Deal>;
        getStages(): Promise<DealStage[]>;
        createStage(name: string, color: string): Promise<DealStage>;
        updateStage(id: string, data: Partial<DealStage>): Promise<DealStage>;
        deleteStage(id: string): Promise<void>;
        reorderStages(ids: string[]): Promise<void>;
    };
    tags: {
        list(): Promise<Tag[]>;
        create(name: string, color: string): Promise<Tag>;
        delete(id: string): Promise<void>;
        addToContact(contactId: string, tagId: string): Promise<void>;
        removeFromContact(contactId: string, tagId: string): Promise<void>;
        getForContact(contactId: string): Promise<Tag[]>;
    };
    settings: {
        get<T>(key: string): Promise<T>;
        set(key: string, value: unknown): Promise<void>;
    };
    shortcuts: {
        list(): Promise<UserShortcut[]>;
        update(action: string, keys: string): Promise<UserShortcut>;
        reset(): Promise<UserShortcut[]>;
    };
    system: {
        getVersion(): Promise<string>;
        openExternal(url: string): Promise<void>;
        showOpenDialog(options: Electron.OpenDialogOptions): Promise<string | null>;
        showSaveDialog(options: Electron.SaveDialogOptions): Promise<string | null>;
    };
}
```

### Preload Script

```typescript
// electron/preload.ts
import { contextBridge, ipcRenderer } from 'electron';

const api: ElectronAPI = {
    contacts: {
        list: (filter) => ipcRenderer.invoke('contacts:list', filter),
        get: (id) => ipcRenderer.invoke('contacts:get', id),
        create: (data) => ipcRenderer.invoke('contacts:create', data),
        update: (id, data) => ipcRenderer.invoke('contacts:update', id, data),
        delete: (id) => ipcRenderer.invoke('contacts:delete', id),
        archive: (id) => ipcRenderer.invoke('contacts:archive', id),
        restore: (id) => ipcRenderer.invoke('contacts:restore', id),
        search: (query, limit) => ipcRenderer.invoke('contacts:search', query, limit),
        importCSV: (filePath, mapping) => ipcRenderer.invoke('contacts:importCSV', filePath, mapping),
        exportCSV: (filter) => ipcRenderer.invoke('contacts:exportCSV', filter),
    },
    deals: {
        list: (filter) => ipcRenderer.invoke('deals:list', filter),
        get: (id) => ipcRenderer.invoke('deals:get', id),
        create: (data) => ipcRenderer.invoke('deals:create', data),
        update: (id, data) => ipcRenderer.invoke('deals:update', id, data),
        delete: (id) => ipcRenderer.invoke('deals:delete', id),
        moveStage: (id, stageId) => ipcRenderer.invoke('deals:moveStage', id, stageId),
        getStages: () => ipcRenderer.invoke('deals:getStages'),
        createStage: (name, color) => ipcRenderer.invoke('deals:createStage', name, color),
        updateStage: (id, data) => ipcRenderer.invoke('deals:updateStage', id, data),
        deleteStage: (id) => ipcRenderer.invoke('deals:deleteStage', id),
        reorderStages: (ids) => ipcRenderer.invoke('deals:reorderStages', ids),
    },
    tags: {
        list: () => ipcRenderer.invoke('tags:list'),
        create: (name, color) => ipcRenderer.invoke('tags:create', name, color),
        delete: (id) => ipcRenderer.invoke('tags:delete', id),
        addToContact: (contactId, tagId) => ipcRenderer.invoke('tags:addToContact', contactId, tagId),
        removeFromContact: (contactId, tagId) => ipcRenderer.invoke('tags:removeFromContact', contactId, tagId),
        getForContact: (contactId) => ipcRenderer.invoke('tags:getForContact', contactId),
    },
    settings: {
        get: (key) => ipcRenderer.invoke('settings:get', key),
        set: (key, value) => ipcRenderer.invoke('settings:set', key, value),
    },
    shortcuts: {
        list: () => ipcRenderer.invoke('shortcuts:list'),
        update: (action, keys) => ipcRenderer.invoke('shortcuts:update', action, keys),
        reset: () => ipcRenderer.invoke('shortcuts:reset'),
    },
    system: {
        getVersion: () => ipcRenderer.invoke('system:getVersion'),
        openExternal: (url) => ipcRenderer.invoke('system:openExternal', url),
        showOpenDialog: (options) => ipcRenderer.invoke('system:showOpenDialog', options),
        showSaveDialog: (options) => ipcRenderer.invoke('system:showSaveDialog', options),
    },
};

contextBridge.exposeInMainWorld('api', api);
```

### Main Process Handler Example

```typescript
// electron/ipc/contacts.ts
import { ipcMain } from 'electron';
import Database from 'better-sqlite3';
import { ulid } from 'ulid';

export function registerContactHandlers(db: Database.Database) {
    ipcMain.handle('contacts:list', (_event, filter: ContactFilter) => {
        let query = 'SELECT * FROM contacts WHERE archived_at IS NULL';
        const params: any[] = [];

        if (filter.company) {
            query += ' AND company = ?';
            params.push(filter.company);
        }
        if (filter.tag) {
            query += ' AND id IN (SELECT contact_id FROM contact_tags WHERE tag_id = ?)';
            params.push(filter.tag);
        }

        query += ' ORDER BY created_at DESC';

        if (filter.limit) {
            query += ' LIMIT ?';
            params.push(filter.limit);
        }

        return db.prepare(query).all(...params);
    });

    ipcMain.handle('contacts:create', (_event, data: CreateContact) => {
        const id = ulid();
        const now = new Date().toISOString();

        db.prepare(`
            INSERT INTO contacts (id, name, email, phone, company, title, notes, custom_fields, created_at, updated_at)
            VALUES (?, ?, ?, ?, ?, ?, ?, ?, ?, ?)
        `).run(id, data.name, data.email, data.phone, data.company, data.title, data.notes,
               JSON.stringify(data.customFields || {}), now, now);

        return db.prepare('SELECT * FROM contacts WHERE id = ?').get(id);
    });

    ipcMain.handle('contacts:search', (_event, query: string, limit = 50) => {
        return db.prepare(`
            SELECT c.*, rank
            FROM contacts_fts
            JOIN contacts c ON contacts_fts.rowid = c.rowid
            WHERE contacts_fts MATCH ?
            ORDER BY rank
            LIMIT ?
        `).all(query + '*', limit);  // prefix matching with *
    });

    // ... remaining handlers: get, update, delete, archive, restore, importCSV, exportCSV
}
```

### Renderer-Side Typed Wrapper

```typescript
// src/lib/ipc.ts
declare global {
    interface Window {
        api: ElectronAPI;
    }
}

export const api = window.api;
```

---

## 5. State Management

### Contact Store

```typescript
// src/stores/contactStore.ts
import { create } from 'zustand';
import { api } from '../lib/ipc';

interface ContactFilter {
    search?: string;
    company?: string;
    tag?: string;
    archived?: boolean;
    limit?: number;
}

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
    setFilter: (filter: Partial<ContactFilter>) => void;
    searchContacts: (query: string) => Promise<Contact[]>;
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

    setFilter: (filter) => {
        set((s) => ({ filter: { ...s.filter, ...filter } }));
        get().loadContacts();
    },

    searchContacts: (query) => api.contacts.search(query),
}));
```

### Deal Store

```typescript
// src/stores/dealStore.ts
import { create } from 'zustand';
import { api } from '../lib/ipc';

interface DealStore {
    deals: Deal[];
    stages: DealStage[];
    selectedId: string | null;
    loading: boolean;

    loadDeals: () => Promise<void>;
    loadStages: () => Promise<void>;
    selectDeal: (id: string | null) => void;
    createDeal: (data: CreateDeal) => Promise<Deal>;
    updateDeal: (id: string, data: UpdateDeal) => Promise<void>;
    deleteDeal: (id: string) => Promise<void>;
    moveToStage: (dealId: string, stageId: string) => Promise<void>;
    getDealsByStage: (stageId: string) => Deal[];
    getStageTotalValue: (stageId: string) => number;
}

export const useDealStore = create<DealStore>((set, get) => ({
    deals: [],
    stages: [],
    selectedId: null,
    loading: false,

    loadDeals: async () => {
        set({ loading: true });
        const deals = await api.deals.list({});
        set({ deals, loading: false });
    },

    loadStages: async () => {
        const stages = await api.deals.getStages();
        set({ stages });
    },

    selectDeal: (id) => set({ selectedId: id }),

    createDeal: async (data) => {
        const deal = await api.deals.create(data);
        set((s) => ({ deals: [...s.deals, deal] }));
        return deal;
    },

    updateDeal: async (id, data) => {
        const updated = await api.deals.update(id, data);
        set((s) => ({
            deals: s.deals.map((d) => (d.id === id ? updated : d)),
        }));
    },

    deleteDeal: async (id) => {
        await api.deals.delete(id);
        set((s) => ({
            deals: s.deals.filter((d) => d.id !== id),
            selectedId: s.selectedId === id ? null : s.selectedId,
        }));
    },

    moveToStage: async (dealId, stageId) => {
        const updated = await api.deals.moveStage(dealId, stageId);
        set((s) => ({
            deals: s.deals.map((d) => (d.id === dealId ? updated : d)),
        }));
    },

    getDealsByStage: (stageId) => get().deals.filter((d) => d.stage_id === stageId),

    getStageTotalValue: (stageId) =>
        get()
            .deals.filter((d) => d.stage_id === stageId)
            .reduce((sum, d) => sum + (d.value || 0), 0),
}));
```

### UI Store

```typescript
// src/stores/uiStore.ts
import { create } from 'zustand';

type View = 'contacts' | 'deals' | 'settings';

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

---

## 6. Keyboard Shortcut System

### Full Shortcut Map

| Key | Action | Context | Customizable |
|-----|--------|---------|-------------|
| `Mod+K` | Open command palette | Global | Yes |
| `Mod+N` | New contact (or new deal on deals view) | Global | Yes |
| `Mod+F` | Focus search bar | Global | Yes |
| `Mod+,` | Open settings | Global | Yes |
| `Mod+/` | Toggle keyboard help | Global | Yes |
| `Escape` | Close modal / deselect / blur search | Global | No |
| `g c` | Go to contacts | Global (chord) | Yes |
| `g d` | Go to deals | Global (chord) | Yes |
| `g s` | Go to settings | Global (chord) | Yes |
| `j` | Select next item in list | List | Yes |
| `k` | Select previous item in list | List | Yes |
| `Enter` | Open selected item | List | No |
| `x` | Toggle select current item | List | Yes |
| `Shift+X` | Select range (from last selected) | List | Yes |
| `e` | Edit selected contact | Contacts | Yes |
| `d d` | Delete selected (with confirmation) | Contacts | Yes |
| `t` | Add tag to selected | Contacts | Yes |
| `n` | Add note to selected | Contacts | Yes |
| `a` | Archive selected contact | Contacts | Yes |
| `n` | New deal | Deals | Yes |
| `m` | Move deal to stage (opens picker) | Deals | Yes |
| `v` | Edit deal value | Deals | Yes |

`Mod` = Cmd on Mac, Ctrl on Windows/Linux.

### Chord Detection

Chords are multi-key sequences pressed within 500ms. For example, `g c` means press `g`, then `c` within 500ms.

```typescript
// src/lib/keyboard.ts

type ShortcutHandler = () => void;

interface ShortcutEntry {
    keys: string;           // 'mod+k', 'g c', 'j', 'd d'
    action: string;         // 'command_palette.toggle'
    context: string;        // 'global', 'contacts', 'deals'
    handler: ShortcutHandler;
}

class KeyboardEngine {
    private shortcuts: ShortcutEntry[] = [];
    private pendingChord: string | null = null;
    private chordTimer: ReturnType<typeof setTimeout> | null = null;
    private activeContext: string = 'global';

    register(entry: ShortcutEntry) {
        this.shortcuts.push(entry);
    }

    setContext(context: string) {
        this.activeContext = context;
    }

    handleKeyDown(event: KeyboardEvent) {
        // Skip if user is typing in an input
        if (this.isInputFocused(event)) return;

        const key = this.normalizeKey(event);

        // Check if this completes a pending chord
        if (this.pendingChord) {
            const chord = `${this.pendingChord} ${key}`;
            this.clearChord();

            const match = this.findMatch(chord);
            if (match) {
                event.preventDefault();
                match.handler();
                return;
            }
        }

        // Check for direct match
        const directMatch = this.findMatch(key);
        if (directMatch) {
            event.preventDefault();
            directMatch.handler();
            return;
        }

        // Check if this could start a chord
        const couldBeChord = this.shortcuts.some((s) =>
            s.keys.startsWith(key + ' ')
        );
        if (couldBeChord) {
            event.preventDefault();
            this.pendingChord = key;
            this.chordTimer = setTimeout(() => this.clearChord(), 500);
        }
    }

    private findMatch(keys: string): ShortcutEntry | undefined {
        return this.shortcuts.find(
            (s) => s.keys === keys &&
            (s.context === 'global' || s.context === this.activeContext)
        );
    }

    private normalizeKey(event: KeyboardEvent): string {
        const parts: string[] = [];
        if (event.metaKey || event.ctrlKey) parts.push('mod');
        if (event.shiftKey) parts.push('shift');
        if (event.altKey) parts.push('alt');
        parts.push(event.key.toLowerCase());
        return parts.join('+');
    }

    private isInputFocused(event: KeyboardEvent): boolean {
        const target = event.target as HTMLElement;
        const tag = target.tagName;
        // Allow mod+ shortcuts even in inputs
        if (event.metaKey || event.ctrlKey) return false;
        return tag === 'INPUT' || tag === 'TEXTAREA' || target.isContentEditable;
    }

    private clearChord() {
        this.pendingChord = null;
        if (this.chordTimer) clearTimeout(this.chordTimer);
    }
}

export const keyboard = new KeyboardEngine();
```

---

## 7. Search Implementation

### FTS5 Configuration

- **Tokenizer:** `porter unicode61` - Porter stemming handles word variants (running/runs/ran all match "run"). Unicode61 handles accented characters.
- **Indexed columns:** name, email, company, phone, notes
- **Prefix matching:** Query `john*` matches "john", "johnson", "johnny"
- **Ranking:** BM25 algorithm (built into FTS5)

### Search Query

```sql
-- Basic search with prefix matching and BM25 ranking
SELECT c.*, rank
FROM contacts_fts
JOIN contacts c ON contacts_fts.rowid = c.rowid
WHERE contacts_fts MATCH ?
  AND c.archived_at IS NULL
ORDER BY rank
LIMIT 50;

-- The query parameter uses FTS5 syntax:
-- 'john'        -> matches "john" in any column
-- 'john*'       -> prefix match
-- 'john smith'  -> matches both terms (implicit AND)
-- '"john smith"' -> phrase match (exact sequence)
-- 'name:john'   -> matches "john" only in name column
```

### Frontend Debounce

```typescript
// src/hooks/useSearch.ts
import { useState, useCallback, useRef } from 'react';
import { api } from '../lib/ipc';

export function useSearch() {
    const [results, setResults] = useState<Contact[]>([]);
    const [loading, setLoading] = useState(false);
    const timerRef = useRef<ReturnType<typeof setTimeout>>();

    const search = useCallback((query: string) => {
        if (timerRef.current) clearTimeout(timerRef.current);

        if (!query.trim()) {
            setResults([]);
            return;
        }

        timerRef.current = setTimeout(async () => {
            setLoading(true);
            const hits = await api.contacts.search(query + '*', 50);
            setResults(hits);
            setLoading(false);
        }, 150); // 150ms debounce
    }, []);

    return { results, loading, search };
}
```

### Performance Targets

- 100K contacts: <50ms query time
- 500K contacts: <200ms query time
- FTS5 index size: roughly 30-40% of source text size
- Rebuild FTS index: `INSERT INTO contacts_fts(contacts_fts) VALUES('rebuild')` - takes ~2s for 100K rows

---

## 8. Custom Fields Implementation

### Type Definitions

```typescript
// src/types/custom-field.ts

export type CustomFieldType =
    | 'text'
    | 'number'
    | 'date'
    | 'select'
    | 'multi-select'
    | 'url'
    | 'email'
    | 'phone';

export interface CustomFieldDefinition {
    id: string;          // ULID
    name: string;        // Display name, e.g. "LinkedIn URL"
    key: string;         // Storage key, e.g. "linkedin_url"
    type: CustomFieldType;
    options?: string[];  // For select/multi-select
    required?: boolean;
    defaultValue?: unknown;
    position: number;    // Sort order in form
}

// Stored in settings table under key 'custom_field_definitions'
// Value is JSON array of CustomFieldDefinition

// Contact custom_fields column stores:
// { "linkedin_url": "https://...", "deal_value": 5000, "source": "referral" }
```

### Storage Approach

Built-in fields (indexed, fast queries):
- `name`, `email`, `phone`, `company`, `title`, `notes`

Custom fields (JSON blob, flexible):
- Stored as JSON string in `custom_fields` column
- Field definitions stored in `settings` table
- Can query custom fields via `json_extract()`:

```sql
-- Filter contacts by custom field value
SELECT * FROM contacts
WHERE json_extract(custom_fields, '$.source') = 'referral';

-- Sort by custom field
SELECT * FROM contacts
ORDER BY json_extract(custom_fields, '$.deal_value') DESC;
```

### Rendering

```typescript
// src/components/shared/CustomFieldRenderer.tsx

interface Props {
    definition: CustomFieldDefinition;
    value: unknown;
    onChange: (value: unknown) => void;
    readOnly?: boolean;
}

export function CustomFieldRenderer({ definition, value, onChange, readOnly }: Props) {
    switch (definition.type) {
        case 'text':
            return <input type="text" value={value as string ?? ''} onChange={e => onChange(e.target.value)} readOnly={readOnly} />;
        case 'number':
            return <input type="number" value={value as number ?? ''} onChange={e => onChange(Number(e.target.value))} readOnly={readOnly} />;
        case 'date':
            return <input type="date" value={value as string ?? ''} onChange={e => onChange(e.target.value)} readOnly={readOnly} />;
        case 'select':
            return (
                <select value={value as string ?? ''} onChange={e => onChange(e.target.value)} disabled={readOnly}>
                    <option value="">Select...</option>
                    {definition.options?.map(opt => <option key={opt} value={opt}>{opt}</option>)}
                </select>
            );
        case 'url':
            return readOnly
                ? <a href={value as string} target="_blank" rel="noreferrer">{value as string}</a>
                : <input type="url" value={value as string ?? ''} onChange={e => onChange(e.target.value)} />;
        // ... email, phone, multi-select
    }
}
```

---

## 9. CSV Import/Export

### Import Flow

1. User clicks "Import" or presses shortcut
2. System dialog opens to select .csv file
3. PapaParse streams first 100 rows for preview
4. FieldMapper UI: user maps CSV columns to contact fields (name, email, phone, company, custom fields)
5. Validation via zod schemas per field
6. Duplicate detection: check `email` column for existing contacts
7. User chooses: skip duplicates, update duplicates, or import all
8. Batch insert: transactions of 500 rows
9. Progress bar shows completion percentage

### Import IPC Handler

```typescript
// electron/ipc/import-export.ts

ipcMain.handle('contacts:importCSV', async (_event, filePath: string, mapping: FieldMapping) => {
    const fileContent = fs.readFileSync(filePath, 'utf-8');
    const { data, errors } = Papa.parse(fileContent, { header: true, skipEmptyLines: true });

    const results: ImportResult = { imported: 0, skipped: 0, errors: [] };

    const insertStmt = db.prepare(`
        INSERT INTO contacts (id, name, email, phone, company, title, notes, custom_fields, created_at, updated_at)
        VALUES (?, ?, ?, ?, ?, ?, ?, ?, ?, ?)
    `);

    const checkDuplicate = db.prepare('SELECT id FROM contacts WHERE email = ?');

    const batchInsert = db.transaction((rows: any[]) => {
        for (const row of rows) {
            const mapped = applyMapping(row, mapping);

            // Duplicate check
            if (mapped.email && checkDuplicate.get(mapped.email)) {
                results.skipped++;
                continue;
            }

            const id = ulid();
            const now = new Date().toISOString();
            insertStmt.run(
                id, mapped.name, mapped.email, mapped.phone, mapped.company,
                mapped.title, mapped.notes, JSON.stringify(mapped.customFields || {}),
                now, now
            );
            results.imported++;
        }
    });

    // Process in batches of 500
    for (let i = 0; i < data.length; i += 500) {
        const batch = data.slice(i, i + 500);
        batchInsert(batch);
    }

    return results;
});
```

### Export

```typescript
ipcMain.handle('contacts:exportCSV', async (_event, filter?: ContactFilter) => {
    const contacts = db.prepare('SELECT * FROM contacts WHERE archived_at IS NULL').all();
    const csv = Papa.unparse(contacts.map(c => ({
        ...c,
        custom_fields: undefined, // flatten custom fields into columns
        ...JSON.parse(c.custom_fields || '{}'),
    })));

    const { filePath } = await dialog.showSaveDialog({
        defaultPath: `contacts-export-${new Date().toISOString().slice(0, 10)}.csv`,
        filters: [{ name: 'CSV', extensions: ['csv'] }],
    });

    if (filePath) {
        fs.writeFileSync(filePath, csv, 'utf-8');
    }

    return filePath;
});
```

---

## 10. Deals Pipeline

### Kanban Board Architecture

```typescript
// src/components/deals/DealBoard.tsx

interface DealBoardProps {
    // No props - reads from dealStore
}

// Uses @dnd-kit/core for drag-and-drop
// Each DealColumn is a droppable zone
// Each DealCard is a draggable item

// On drop: call dealStore.moveToStage(dealId, newStageId)
// Optimistic update: move card in UI immediately, sync to SQLite
```

### Default Stages

| Position | Stage | Color | Description |
|----------|-------|-------|-------------|
| 0 | Lead | Gray (#6B7280) | Initial contact, no qualification |
| 1 | Qualified | Blue (#3B82F6) | Confirmed budget and need |
| 2 | Proposal | Purple (#8B5CF6) | Proposal sent |
| 3 | Negotiation | Yellow (#F59E0B) | Terms being negotiated |
| 4 | Closed Won | Green (#10B981) | Deal closed successfully |
| 5 | Closed Lost | Red (#EF4444) | Deal lost |

### Deal Card Display

Each card in the kanban shows:
- Deal title (bold)
- Value formatted as currency ($5,000)
- Linked contact name (if any)
- Days in current stage (calculated from `updated_at`)
- Colored left border matching stage color

### Column Header

Each column header shows:
- Stage name
- Deal count in stage
- Total value in stage (e.g. "$45,000")

### Custom Stages

Users can:
- Add new stages (name + color picker)
- Rename existing stages
- Reorder stages (drag column headers)
- Delete stages (must move deals first)
- Cannot delete "Closed Won" or "Closed Lost" (required)

---

## 11. Component Architecture

### ContactList

```typescript
interface ContactListProps {
    // Reads from contactStore
}

// Key behaviors:
// - Virtual scrolling via @tanstack/react-virtual
// - Renders ContactRow for each visible item
// - Handles j/k keyboard navigation
// - Supports multi-select with x/Shift+X
// - Fixed row height: 64px (name + email + company)
// - Estimated total size enables scrollbar sizing

// Performance:
// - Only renders visible rows + 5 overscan
// - 100K contacts scrolls at 60fps
// - Row click selects, Enter opens detail
```

### ContactDetail

```typescript
interface ContactDetailProps {
    contactId: string;
}

// Key behaviors:
// - Displays all built-in fields (read-only by default)
// - Displays custom fields via CustomFieldRenderer
// - Shows linked deals (list with stage + value)
// - Shows tags as colored chips
// - Edit mode toggled by 'e' key or edit button
// - Delete with 'd d' (shows confirmation dialog)
// - Notes section with plaintext editing
```

### DealBoard

```typescript
// Key behaviors:
// - Horizontal scroll if many stages
// - Each column is a DndKit droppable
// - Cards are DndKit draggable
// - Drop triggers moveToStage IPC call
// - Optimistic UI update (move card immediately)
// - Column headers show count + total value
// - Click card to open DealForm in side panel
```

### CommandPalette

```typescript
// Key behaviors:
// - Opens with Mod+K
// - Fuzzy search across all actions
// - Actions: navigate, create contact, create deal, open settings, search, import/export
// - Recent actions shown first
// - Arrow keys to navigate, Enter to select, Esc to close
// - Built on cmdk library

// Action registry:
const actions: CommandAction[] = [
    { id: 'nav.contacts', label: 'Go to Contacts', shortcut: 'g c', handler: () => navigate('contacts') },
    { id: 'nav.deals', label: 'Go to Deals', shortcut: 'g d', handler: () => navigate('deals') },
    { id: 'contact.create', label: 'New Contact', shortcut: 'Mod+N', handler: () => openCreateContact() },
    { id: 'deal.create', label: 'New Deal', handler: () => openCreateDeal() },
    { id: 'import', label: 'Import CSV', handler: () => openImport() },
    { id: 'export', label: 'Export Contacts', handler: () => exportContacts() },
    // ... more actions
];
```

---

## 12. Build and Distribution

### electron-builder.yml

```yaml
appId: com.localcrm.app
productName: LocalCRM
copyright: Copyright 2026

directories:
  output: dist
  buildResources: resources

files:
  - "dist-electron/**/*"
  - "dist/**/*"

mac:
  category: public.app-category.business
  target:
    - target: dmg
      arch:
        - universal
  hardenedRuntime: true
  entitlements: build/entitlements.mac.plist
  entitlementsInherit: build/entitlements.mac.plist
  notarize: true

win:
  target:
    - target: nsis
      arch:
        - x64
  certificateFile: ${env.WIN_CSC_LINK}
  certificatePassword: ${env.WIN_CSC_KEY_PASSWORD}

linux:
  target:
    - AppImage
    - deb
  category: Office

nsis:
  oneClick: false
  allowToChangeInstallationDirectory: true

publish:
  provider: github
  releaseType: release
```

### Auto-Updater

```typescript
// electron/main.ts
import { autoUpdater } from 'electron-updater';

autoUpdater.autoDownload = true;
autoUpdater.autoInstallOnAppQuit = true;

autoUpdater.on('update-available', () => {
    // Notify renderer that update is downloading
});

autoUpdater.on('update-downloaded', () => {
    // Notify renderer, prompt to restart
});

// Check for updates on launch and every 4 hours
autoUpdater.checkForUpdates();
setInterval(() => autoUpdater.checkForUpdates(), 4 * 60 * 60 * 1000);
```

### GitHub Actions CI/CD

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
        os: [macos-latest, windows-latest, ubuntu-latest]
    runs-on: ${{ matrix.os }}
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 22
      - run: npm ci
      - run: npm run build
      - name: Build Electron
        env:
          GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          CSC_LINK: ${{ secrets.MAC_CSC_LINK }}
          CSC_KEY_PASSWORD: ${{ secrets.MAC_CSC_KEY_PASSWORD }}
          WIN_CSC_LINK: ${{ secrets.WIN_CSC_LINK }}
          WIN_CSC_KEY_PASSWORD: ${{ secrets.WIN_CSC_KEY_PASSWORD }}
        run: npx electron-builder --publish always
```

---

## 13. Testing Strategy

### Unit Tests (vitest)

```typescript
// tests/unit/contactStore.test.ts
import { describe, it, expect, vi } from 'vitest';

// Mock IPC
vi.mock('../lib/ipc', () => ({
    api: {
        contacts: {
            list: vi.fn().mockResolvedValue([]),
            create: vi.fn().mockResolvedValue({ id: '1', name: 'Test' }),
        },
    },
}));

describe('contactStore', () => {
    it('adds new contact to list after create', async () => {
        const store = useContactStore.getState();
        await store.createContact({ name: 'Test' });
        expect(store.contacts).toHaveLength(1);
    });

    it('selectNext moves selection down', () => {
        // ...
    });
});
```

### E2E Tests (Playwright)

```typescript
// tests/e2e/contacts.spec.ts
import { test, expect, _electron as electron } from 'playwright';

test('create and search for contact', async () => {
    const app = await electron.launch({ args: ['.'] });
    const page = await app.firstWindow();

    // Create contact
    await page.keyboard.press('Meta+n');
    await page.fill('[data-testid="contact-name"]', 'John Doe');
    await page.fill('[data-testid="contact-email"]', 'john@example.com');
    await page.click('[data-testid="save-contact"]');

    // Search
    await page.keyboard.press('Meta+f');
    await page.fill('[data-testid="search-input"]', 'john');
    await expect(page.locator('[data-testid="search-result"]')).toContainText('John Doe');

    await app.close();
});
```

### Coverage Targets

| Area | Target |
|------|--------|
| Stores (business logic) | 80%+ |
| Utilities (validators, keyboard) | 90%+ |
| Components | 60%+ |
| IPC handlers | 70%+ |
| Overall | 65%+ |

---

## 14. Performance Targets

| Metric | Target | How to Measure |
|--------|--------|---------------|
| Cold start (app ready) | <3.2s | Time from launch to `app.whenReady()` + first render |
| Search (100K contacts) | <50ms | FTS5 query execution time in main process |
| List scroll (10K contacts) | 60fps | Chrome DevTools Performance tab |
| Contact create | <20ms | IPC round-trip (invoke to response) |
| Contact update | <15ms | IPC round-trip |
| CSV import (10K rows) | <5s | Total batch insert time |
| CSV import (100K rows) | <45s | Total batch insert time |
| Memory (idle, empty DB) | <180MB | Electron Task Manager |
| Memory (100K contacts loaded) | <350MB | Electron Task Manager |
| FTS5 index rebuild (100K) | <3s | `INSERT INTO fts(fts) VALUES('rebuild')` |
| Bundle size (installer) | ~165MB | File size of .dmg/.exe |
| Database file (100K contacts) | ~50MB | File size of SQLite DB |

---

## 15. Day-by-Day Implementation Schedule

### Week 1: Foundation + Contacts Core

| Day | Instance 1 | Instance 2 | Instance 3 | Integration |
|-----|-----------|-----------|-----------|-------------|
| 1 | Project setup: Electron + Vite + TypeScript. Configure electron main/preload/renderer. Get blank window rendering React. | Tailwind 4 setup. Base layout components: App shell, Sidebar skeleton, TopBar skeleton. Global CSS variables + dark theme. | SQLite setup with better-sqlite3. Database.ts with connection, WAL mode, foreign keys. Run full schema SQL. Verify tables created. | End of day: merge all 3. App launches with dark themed shell and working SQLite DB. |
| 2 | Contact CRUD IPC handlers: list, get, create, update, delete. Register in main.ts. Preload exposure for contacts namespace. | ContactList component with @tanstack/react-virtual. ContactRow component. Hard-coded mock data to verify scrolling. 64px row height. | ContactDetail component (read-only view). ContactForm component (create/edit). Zod validation schemas for contact fields. | Merge. Wire ContactList to real IPC. Verify create/read/update/delete works end-to-end. |
| 3 | FTS5 virtual table creation. Sync triggers (insert/update/delete). Search IPC handler with prefix matching and BM25 ranking. | SearchBar component in TopBar. SearchResults dropdown. Wire to search IPC. 150ms debounce. | Zustand stores: contactStore (full implementation), uiStore (view, sidebar, modals). Wire stores to components. | Merge. Test search works with real data. Type "john", see matching contacts. |
| 4 | Tags: create tags table, contact_tags junction. Tag CRUD IPC handlers. addToContact/removeFromContact handlers. | TagInput component: autocomplete dropdown, colored chips, create-on-type. Wire to tag IPC. | Custom fields: settings storage for field definitions. CustomFieldRenderer component for all 8 field types. | Merge. Create contacts with tags and custom fields. Verify persistence. |
| 5 | Archive/restore IPC handlers. Contact filtering by tag, company, archived status. List handler supports all filter params. | Sidebar: navigation between views (contacts, deals, settings). Active state highlighting. Collapse/expand. Counts per section. | Keyboard engine (keyboard.ts): chord detection, context routing, isInputFocused guard. Register global shortcuts (Mod+K, Mod+N, Escape). | Full integration test. Create 1000 contacts via script, test scroll performance, search, filtering. Fix any issues. |

### Week 2: Features Sprint (Full Parallel)

| Day | Instance 1 | Instance 2 | Instance 3 | Integration |
|-----|-----------|-----------|-----------|-------------|
| 6 | Custom field definitions management UI. Add/edit/delete/reorder field definitions. Stored in settings table. | CSV import: papaparse integration. Import IPC handler with batch transactions (500 per batch). Duplicate detection on email. | Command palette (cmdk): action registry, fuzzy search, recent actions. Render with proper styling. | - |
| 7 | Custom field filtering: add custom field filters to contact list. json_extract queries. Sort by custom field. | CSVImport component: file picker, preview table (first 10 rows), FieldMapper (dropdown per column). Progress bar. | Global shortcuts: Mod+F (focus search), Mod+N (new contact), Mod+, (settings). j/k list navigation. | - |
| 8 | Deals database: deal_stages + deals tables (already in schema). Deal CRUD IPC handlers. Stage management handlers. | CSV export IPC handler. Export all or filtered contacts. Include custom fields as flat columns. Save dialog. | Chord shortcuts: g+c (contacts), g+d (deals), g+s (settings). d+d (delete with confirm). Context-aware routing. | - |
| 9 | DealBoard component: horizontal scroll container. DealColumn: droppable zone + value summary header. DealCard: title, value, contact, days in stage. | Import error handling: validation errors per row, skip/retry options, final summary report. | Keyboard help modal: table of all shortcuts grouped by context. Toggle with Mod+/. Highlight customizable shortcuts. | - |
| 10 | DealForm: create/edit deal. Stage selector, value input, contact linker (search contacts), notes, custom fields. | Import duplicate strategies: skip, update, create anyway. User selection UI before import starts. | Focus management: react-focus-lock on all modals (CommandPalette, KeyboardHelp, DealForm, ContactForm). Tab trapping. | Full merge. Test all features end-to-end. Fix integration issues. Run through full user flow. |

### Week 3: Deals + Polish

| Day | Instance 1 | Instance 2 | Instance 3 | Integration |
|-----|-----------|-----------|-----------|-------------|
| 11 | Drag-and-drop: @dnd-kit/core integration. DealCard draggable, DealColumn droppable. Optimistic UI update on drop. moveStage IPC. | Contact-deal linking: show deals on ContactDetail. Create deal from contact view. Filter deals by contact. | Shortcut persistence: user_shortcuts table CRUD. Custom shortcut editor in settings. Reset to defaults button. | - |
| 12 | Deal filtering sidebar: filter by stage, value range (min/max), date range (created, closed). Filter badge counts. | Pipeline value summaries: per-column total, grand total, average deal value, conversion rate (won/total). | Settings page: theme toggle (dark/light/system), sidebar default state, default currency, CSV delimiter preference. | - |
| 13 | Deal custom fields: same CustomFieldRenderer, separate definitions from contacts. Deal-specific field types (currency, percentage). | Contact archive: bulk archive/restore. Archive confirmation. Archived contacts view (separate tab or filter). | Error boundaries: React ErrorBoundary at route level. Graceful fallback UI. Error logging to file. IPC error handling patterns. | - |
| 14 | - | - | - | Full integration day. Merge ALL parallel streams. Resolve conflicts. E2E test every user flow. Performance profiling with 10K contacts + 500 deals. |
| 15 | Memory leak detection: long-running test, monitor Electron process memory. Fix leaks in virtual scroll, store subscriptions. | Performance optimization: analyze slow queries with EXPLAIN. Add missing indexes if needed. Optimize hot paths. | Bug fixes from Day 14 testing. Edge cases: empty states, very long names, special characters, RTL text. | Final integration. All features working together. |

### Week 4: Distribution + Beta

| Day | Instance 1 | Instance 2 | Instance 3 | Integration |
|-----|-----------|-----------|-----------|-------------|
| 16 | electron-builder Mac build. DMG creation. Universal binary (arm64 + x64). Test on Intel and Apple Silicon Macs. | Windows build. NSIS installer. Test on Windows 10 and 11. Handle Windows Defender warnings. | Linux build. AppImage + .deb package. Test on Ubuntu 22.04 and 24.04. | All 3 platform builds ready. |
| 17 | Mac code signing ($99 Apple Developer). Notarization with notarytool. Verify Gatekeeper passes. | Windows code signing (DigiCert certificate). Sign installer and executable. Verify SmartScreen passes. | Auto-updater: electron-updater + GitHub Releases. Test update flow (v0.0.1 -> v0.0.2). Verify download + install. | Signed builds for Mac + Windows. |
| 18 | GitHub Actions CI/CD: matrix build (mac, win, linux). Auto-publish to GitHub Releases on tag push. | Performance test suite: benchmark script that creates 100K contacts, runs search queries, measures FTS5 times, scroll FPS. | E2E test suite: Playwright tests for critical flows (create contact, search, import CSV, create deal, drag deal, keyboard shortcuts). | CI pipeline green. All tests pass. |
| 19 | Alpha release build (v0.1.0). Create GitHub Release with Mac + Windows + Linux binaries. Write release notes. | Outreach: cold-message 20 consultants on LinkedIn. Prepare demo script. Record 2-minute demo video. | Landing page: simple page with download links, feature highlights, pricing ($50/month). Stripe integration for future billing. | Alpha in the hands of 20 people. |
| 20 | Buffer day. Bug fixes from alpha feedback. Crash reports. UI papercuts. | Performance issues from real-world usage. Large contact databases. Slow machines. | Beta release prep. Update version. Final polish. Announce on relevant communities. | Beta release at $50/month. |

---

## 16. Phase 2 Preview

### Week 5-6: Cloud Sync (PowerSync)
- Set up Postgres backend (Supabase or Railway)
- Integrate PowerSync SDK for SQLite-to-Postgres sync
- Implement sync rules (per-user data isolation)
- Conflict resolution strategy: last-write-wins with timestamps
- Sync status indicator in UI (synced, syncing, offline)
- Multi-device support: same account on multiple machines

### Week 7-8: Email Integration
- IMAP/SMTP integration for reading/sending email
- Conversation threads linked to contacts
- Inbox view with email list and compose
- Auto-link emails to contacts by email address
- Email templates for common outreach

### Week 9-10: Refinement
- Analytics dashboard: pipeline metrics, conversion rates, revenue tracking
- Reporting: export pipeline reports, activity summaries
- API: REST API for integrations (Zapier, Make)
- Mobile companion app evaluation (React Native or PWA)

### Ongoing
- User feedback iteration
- Performance monitoring
- Additional integrations (calendar, social media)
- Team features (shared contacts, permissions) for agency tier
