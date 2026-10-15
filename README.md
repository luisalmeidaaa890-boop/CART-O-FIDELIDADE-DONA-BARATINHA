# CART-O-FIDELIDADE-DONA-BARATINHA
export default {
  logoUrl:
    "https://files.manuscdn.com/user_upload_by_module/session_file/310519663973705707/NTLbBQxXkETByZtU.png",
};
import { useEffect, useState } from "react";
import { useLocation } from "wouter";
import { AppShell, LogoMark } from "@/components/AppShell";
import Login from "@/pages/Login";
import {
  AdminCustomerProfile,
  AdminCustomers,
  AdminDashboard,
  AdminNotifications,
  AdminPurchases,
  AdminSettings,
  CustomerDashboard,
  CustomerProfile,
  CustomerPurchases,
} from "@/pages/Portal";
import { Privacy, Terms } from "@/pages/Legal";
import { api } from "@/lib/api";

export default function App() {
  const [profile, setProfile] = useState<any | null>(null);
  const [checking, setChecking] = useState(true);
  const [location, setLocation] = useLocation();

  useEffect(() => {
    api
      .me()
      .then(setProfile)
      .catch(() => setProfile(null))
      .finally(() => setChecking(false));
  }, []);

  function navigate(path: string) {
    setLocation(path);
    window.scrollTo({ top: 0, behavior: "smooth" });
  }

  async function logout() {
    await api.logout().catch(() => undefined);
    setProfile(null);
    navigate("/login");
  }

  if (checking)
    return (
      <div className="app-loading">
        <LogoMark />
        <p>Preparando seu Clube…</p>
      </div>
    );
  if (location === "/privacidade")
    return (
      <PublicPage
        onBack={() =>
          navigate(
            profile
              ? profile.auth.role === "admin"
                ? "/admin"
                : "/cliente"
              : "/login"
          )
        }
      >
        <Privacy />
      </PublicPage>
    );
  if (location === "/termos")
    return (
      <PublicPage
        onBack={() =>
          navigate(
            profile
              ? profile.auth.role === "admin"
                ? "/admin"
                : "/cliente"
              : "/login"
          )
        }
      >
        <Terms />
      </PublicPage>
    );
  if (!profile) return <Login onLoggedIn={setProfile} />;

  const isAdmin = profile.auth.role === "admin";
  const home = isAdmin ? "/admin" : "/cliente";
  const active = location.startsWith("/admin/clientes/")
    ? "/admin/clientes"
    : location.startsWith("/admin")
      ? location
      : location.startsWith("/cliente")
        ? location
        : home;
  let content: React.ReactNode;
  if (isAdmin) {
    if (location === "/admin/clientes")
      content = <AdminCustomers onNavigate={navigate} />;
    else if (location.startsWith("/admin/clientes/"))
      content = (
        <AdminCustomerProfile
          id={Number(location.split("/").pop())}
          onBack={() => navigate("/admin/clientes")}
        />
      );
    else if (location === "/admin/compras") content = <AdminPurchases />;
    else if (location === "/admin/notificacoes")
      content = <AdminNotifications />;
    else if (location === "/admin/configuracoes") content = <AdminSettings />;
    else content = <AdminDashboard />;
  } else {
    if (location === "/cliente/compras")
      content = <CustomerPurchases profile={profile} />;
    else if (location === "/cliente/perfil")
      content = <CustomerProfile profile={profile} />;
    else content = <CustomerDashboard profile={profile} />;
  }
  return (
    <AppShell
      role={isAdmin ? "admin" : "customer"}
      userName={profile.auth.name}
      active={active}
      onNavigate={navigate}
      onLogout={logout}
    >
      {content}
      <footer className="app-footer">
        <span>Clube de Fidelidade</span>
        <span>Privacidade · Termos</span>
      </footer>
    </AppShell>
  );
}

function PublicPage({
  children,
  onBack,
}: {
  children: React.ReactNode;
  onBack: () => void;
}) {
  return (
    <div className="public-page">
      <header className="public-header">
        <button onClick={onBack} className="back-link">
          ← Voltar
        </button>
        <div className="flex items-center gap-3">
          <LogoMark />
          <span className="font-display font-extrabold">Clube Fidelidade</span>
        </div>
      </header>
      <main className="page-wrap">{children}</main>
    </div>
  );
}
import {
  Bell,
  BookOpen,
  ChevronRight,
  CircleUserRound,
  Gift,
  LayoutDashboard,
  LogOut,
  Menu,
  Settings,
  ShoppingBag,
  Sparkles,
  Users,
  X,
} from "lucide-react";
import { useState } from "react";

export function LogoMark({ className = "" }: { className?: string }) {
  return (
    <div className={`brand-mark ${className}`}>
      <img src="/logo.png" alt="Dona Baratinha" />
    </div>
  );
}

export function Brand({ compact = false }: { compact?: boolean }) {
  return (
    <div className="flex items-center gap-3">
      <LogoMark />
      {!compact && (
        <div>
          <p className="text-[10px] font-extrabold uppercase tracking-[0.24em] text-blue-600">
            Clube
          </p>
          <p className="font-display text-lg font-extrabold leading-none text-slate-900">
            Fidelidade
          </p>
        </div>
      )}
    </div>
  );
}

export function Badge({
  children,
  tone = "blue",
}: {
  children: React.ReactNode;
  tone?: "blue" | "green" | "yellow" | "coral" | "slate";
}) {
  return <span className={`status-badge status-${tone}`}>{children}</span>;
}

export function ProgressBar({ percentage }: { percentage: number }) {
  return (
    <div className="progress-track">
      <div
        className="progress-fill"
        style={{ width: `${Math.min(100, Math.max(0, percentage))}%` }}
      />
    </div>
  );
}

export function MetricCard({
  icon: Icon,
  label,
  value,
  hint,
  tone = "blue",
}: {
  icon: React.ElementType;
  label: string;
  value: string | number;
  hint?: string;
  tone?: string;
}) {
  return (
    <div className={`metric-card tone-${tone}`}>
      <div className="flex items-start justify-between gap-3">
        <div>
          <p className="eyebrow">{label}</p>
          <p className="metric-value">{value}</p>
          {hint && <p className="mt-1 text-xs text-slate-500">{hint}</p>}
        </div>
        <div className="metric-icon">
          <Icon size={20} />
        </div>
      </div>
    </div>
  );
}

export function AppShell({
  children,
  role,
  userName,
  active,
  onNavigate,
  onLogout,
}: {
  children: React.ReactNode;
  role: "admin" | "customer";
  userName: string;
  active: string;
  onNavigate: (path: string) => void;
  onLogout: () => void;
}) {
  const [open, setOpen] = useState(false);
  const customerLinks = [
    { path: "/cliente", label: "Meu progresso", icon: LayoutDashboard },
    { path: "/cliente/compras", label: "Minhas compras", icon: ShoppingBag },
    { path: "/cliente/perfil", label: "Meu perfil", icon: CircleUserRound },
  ];
  const adminLinks = [
    { path: "/admin", label: "Visão geral", icon: LayoutDashboard },
    { path: "/admin/clientes", label: "Clientes", icon: Users },
    { path: "/admin/compras", label: "Compras", icon: ShoppingBag },
    { path: "/admin/notificacoes", label: "Notificações", icon: Bell },
    { path: "/admin/configuracoes", label: "Configurações", icon: Settings },
  ];
  const links = role === "admin" ? adminLinks : customerLinks;
  return (
    <div className="min-h-screen bg-[#f7f6f2] text-slate-900">
      <aside className={`sidebar ${open ? "sidebar-open" : ""}`}>
        <div className="flex items-center justify-between px-6 py-6">
          <Brand />
          <button
            className="icon-button md:hidden"
            onClick={() => setOpen(false)}
          >
            <X size={18} />
          </button>
        </div>
        <div className="px-4 pb-5">
          <div className="user-chip">
            <div className="avatar">{userName.slice(0, 1).toUpperCase()}</div>
            <div className="min-w-0">
              <p className="truncate text-sm font-bold">{userName}</p>
              <p className="text-xs text-slate-400">
                {role === "admin" ? "Equipe administrativa" : "Membro do clube"}
              </p>
            </div>
          </div>
        </div>
        <nav className="space-y-1 px-3">
          {links.map(link => {
            const Icon = link.icon;
            return (
              <button
                key={link.path}
                onClick={() => {
                  onNavigate(link.path);
                  setOpen(false);
                }}
                className={`nav-link ${active === link.path ? "nav-link-active" : ""}`}
              >
                <Icon size={18} />
                <span>{link.label}</span>
                {active === link.path && (
                  <ChevronRight size={15} className="ml-auto" />
                )}
              </button>
            );
          })}
        </nav>
        <div className="mt-auto px-3 pb-5">
          <button
            onClick={onLogout}
            className="nav-link w-full text-slate-500 hover:text-rose-600"
          >
            <LogOut size={18} />
            <span>Sair do clube</span>
          </button>
        </div>
      </aside>
      <main className="md:ml-[260px]">
        <header className="topbar">
          <button
            className="icon-button md:hidden"
            onClick={() => setOpen(true)}
          >
            <Menu size={20} />
          </button>
          <div>
            <p className="eyebrow hidden sm:block">
              {role === "admin" ? "Central de gestão" : "Seu cartão virtual"}
            </p>
            <p className="font-display text-lg font-extrabold text-slate-900">
              {role === "admin" ? (
                "Painel administrativo"
              ) : (
                <>Olá, {userName}</>
              )}
            </p>
          </div>
          <div className="ml-auto flex items-center gap-3">
            <div className="hidden items-center gap-2 rounded-full bg-white px-3 py-2 text-xs font-semibold text-slate-500 shadow-sm sm:flex">
              <Sparkles size={14} className="text-amber-500" /> Cada compra vale
              um passo
            </div>
            <div className="avatar avatar-sm">
              {userName.slice(0, 1).toUpperCase()}
            </div>
          </div>
        </header>
        <div className="page-wrap">{children}</div>
      </main>
    </div>
  );
}

export function SectionHeading({
  eyebrow,
  title,
  description,
  action,
}: {
  eyebrow?: string;
  title: string;
  description?: string;
  action?: React.ReactNode;
}) {
  return (
    <div className="mb-6 flex flex-col gap-4 sm:flex-row sm:items-end sm:justify-between">
      <div>
        {eyebrow && <p className="eyebrow text-blue-600">{eyebrow}</p>}
        <h1 className="font-display text-3xl font-extrabold tracking-tight text-slate-950 sm:text-4xl">
          {title}
        </h1>
        {description && (
          <p className="mt-2 max-w-2xl text-sm leading-6 text-slate-500">
            {description}
          </p>
        )}
      </div>
      {action}
    </div>
  );
}

export function EmptyState({
  icon: Icon = Gift,
  title,
  description,
}: {
  icon?: React.ElementType;
  title: string;
  description: string;
}) {
  return (
    <div className="empty-state">
      <div className="empty-icon">
        <Icon size={24} />
      </div>
      <h3 className="font-display text-lg font-extrabold">{title}</h3>
      <p className="mt-1 max-w-sm text-sm leading-6 text-slate-500">
        {description}
      </p>
    </div>
  );
}
import { auditLogs } from "../data";

export const AuditService = {
  list: auditLogs,
};
# Default login and project access

When adding login, sign-up, user accounts, or authentication without a specified provider, use Manus OAuth by default. The user does not need to mention Manus. Do not build a separate email/password authentication system or choose another provider unless the user explicitly requests it. Preserve an existing application's authentication unless the user requests a change.

Application login is separate from project visibility. Manus OAuth requires `features.server: true`, without a separate auth feature key; the declaration schema is in [features](features.md). Provider-specific third-party credentials use [secrets](secrets.md).

## "Who can open this URL" is not your login

Project visibility is a Dashboard sharing control, not application configuration. On a non-public project the platform admits the owner, this session's collaborators, and invited users after Manus authentication, before a request reaches the application. A public project has no such platform access gate. Neither setting creates an application session.

## Manus account login: the wire contract

The following values are platform-provided for application authentication.
Managed Cloud development injects them into the application environment; the published deployment receives them too. [Local](../worklocally/SKILL.md) owns Local Host loading. The internal config tool's `GET env` shows redacted names, not usable values.

| Value | Meaning |
| --- | --- |
| `MANUS_PROJECT_ID` | OAuth client ID; public |
| `MANUS_OAUTH_PORTAL_URL` | Browser authorization base; public |
| `MANUS_OAUTH_API_URL` | Token exchange and identity API base |

The browser authorization endpoint is `$MANUS_OAUTH_PORTAL_URL/app-auth` with query fields `appId`, `redirectUri` (absolute callback URL), `state` (base64-encoded JSON containing `redirectUri` and `nonce`), and `responseType=code`.

Token exchange is server-side HTTP POST, JSON:

```text
$MANUS_OAUTH_API_URL/webdev.v1.WebDevAuthPublicService/ExchangeToken
{"clientId":"<MANUS_PROJECT_ID>","grantType":"authorization_code","code":"...","redirectUri":"..."}
```

Successful exchange returns the following camelCase JSON fields:

| Field | Meaning |
| --- | --- |
| `accessToken` | Access token supplied to the subsequent identity lookup |
| `tokenType` | `Bearer` |
| `expiresIn` | Lifetime value in seconds; the current service returns `2147483647` and stores no time-based expiry |
| `refreshToken` | Optional in the response schema; the current exchange supplies one |
| `scope` | The scope stored with the authorization code |
| `idToken` | Current service's opaque ID-token value, not a signed OIDC JWT |

The current exchange implementation accepts only `grantType: "authorization_code"`. A `refreshToken` field does not imply a supported refresh-token grant on this endpoint.

Identity lookup is HTTP POST, JSON:

```text
$MANUS_OAUTH_API_URL/webdev.v1.WebDevAuthPublicService/GetUserInfo
{"accessToken":"..."}
→ {openId, name, email, platforms}
```

The callback is the application's actual frontend URL. Its acceptance is governed by the platform redirect policy; a loopback URL is not automatically permitted. The browser-visible frontend origin can differ from the server's internal host. The platform OAuth exchange does not create an application's own login implementation or session handling.

## Reserved names the platform occupies

`app_session_id` is a platform-owned cookie. On non-public projects the gateway verifies it with the project's signing secret; an application cookie using that name can be rejected by the gateway before application code runs.

The [platform environment contract](service-api.md#platform-owned-compatibility-names) owns
signing-secret names and publication-only compatibility aliases. Use that contract when
choosing environment names for an application's separate session implementation.

The [routing contract](routing-and-responses.md#published-routes) owns reserved path prefixes and interception. Platform OAuth endpoints include `/manus-oauth/callback` and `/manus-oauth/logout`. The storage namespace is owned by [assets and storage](storage.md#stable-asset-addresses), not an application OAuth callback namespace.

## Preview and application sessions

For Web applications using Manus account login, preserve the exact application session cookie name `webdev_app_session` to remain compatible with Preview auto-login. This applies to the default starter and `template: "flexible"`, regardless of framework or backend language. Preview injects a cookie with this fixed name; it does not discover a renamed cookie or read the application's `COOKIE_NAME` variable. Use the same name when setting the normal OAuth session cookie, reading authenticated requests, and clearing the session on logout.

The cookie name alone is insufficient: validate the project JWT using `MANUS_JWT_SECRET` and HS256, check expiry and that `appId` matches `MANUS_PROJECT_ID`, and resolve the authenticated `openId` through the application's user and permission logic. Preserve the starter's validation when adapting it to another stack. Preview auto-login requires support from the Preview host; if no valid session is supplied, keep normal login and the logged-out state. Platform OAuth and the platform access gate alone do not establish the application's own session.

Cloud Preview forwards the application's `Authorization` and `Cookie` headers to the development
server. An application may use its own Bearer token through Preview; forwarding a header does not
authenticate a user or grant a role. Browser cookie delivery still follows the rules below.

Cloud Dashboard embeds the application's public Preview in a cross-site iframe. The public connection is HTTPS even when the development server receives HTTP internally. Browser cookies marked `SameSite=Lax` or `SameSite=Strict` are not sent on that cross-site iframe request. `SameSite=None; Secure` makes a cookie eligible for that context, but browser third-party-cookie restrictions can still block it.

For application-owned login, do not choose `Secure`/`SameSite` from the internal `req.secure` value
or `NODE_ENV`: the HTTPS Preview may reach the app over HTTP while running in development mode.

Partitioned cookies have separate storage contexts for the embedded Preview and a standalone tab. Login in one context therefore does not establish a session in the other. The internal request scheme alone does not describe the browser's HTTPS cookie context. [Runtime](runtime.md) owns the platform proxy and embedding-header contract.

## Authentication implementation

For a new application using Manus OAuth, declare its server capability at creation, before implementing login. Implement application sessions and permissions around the returned Manus identity. Credentials for an explicitly requested third-party provider use the protected-secret declaration path.

Bind OAuth `state` to the initiating browser using a single-use nonce in a short-lived cookie. On callback, verify that state/nonce binding before exchanging the code and fail closed on mismatch. After login, redirect only to a fixed in-application path, never to an arbitrary URL supplied in the request. Preserve the actual frontend callback origin and the platform's redirect-acceptance rules.

Use real login and validated application sessions in Cloud Preview, Manus Studio and the published site. Show logged-out state until it succeeds; do not invent a Preview user, bypass authentication under `NODE_ENV`/another development switch, or defer real login until publishing. If cookies, callback policy or authorization prevent the flow, keep the real implementation and report the specific observed failure.
export const COOKIE_NAME = "webdev_app_session";
export const ONE_YEAR_MS = 1000 * 60 * 60 * 24 * 365;
export const AXIOS_TIMEOUT_MS = 30_000;
export const UNAUTHED_ERR_MSG = 'Please login (10001)';
export const NOT_ADMIN_ERR_MSG = 'You do not have required permission (10002)';

// One-time nonce cookie that binds an OAuth login to the browser that started
// it. The `__Host-` prefix forces the cookie host-only (Secure, Path=/, no
// Domain), so a sibling *.manus.space site cannot plant a matching value in a
// victim's browser.
export const OAUTH_STATE_COOKIE = "__Host-oauth_state";

// `state` carries the callback redirect URI (used at token exchange) plus the
// CSRF nonce. Defined here so the client encoder and server decoder never drift.
export type OAuthState = { redirectUri: string; nonce?: string };

export const encodeOAuthState = (state: OAuthState): string =>
  btoa(JSON.stringify(state));

export const decodeOAuthState = (state: string): OAuthState => {
  let decoded: string;
  try {
    decoded = atob(state);
  } catch {
    // Malformed base64 (e.g. attacker-supplied garbage). Return no nonce so the
    // callback's CSRF guard rejects it with 403 — never throw, since the caller
    // runs outside the request handler's try/catch.
    return { redirectUri: "" };
  }
  try {
    const parsed = JSON.parse(decoded);
    if (parsed && typeof parsed.redirectUri === "string") return parsed;
  } catch {
    // Legacy links: `state` was a bare base64(redirectUri) with no nonce.
  }
  return { redirectUri: decoded };
};
import type { CreateExpressContextOptions } from "@trpc/server/adapters/express";
import type { User } from "../../drizzle/schema";
import { sdk } from "./sdk";

export type TrpcContext = {
  req: CreateExpressContextOptions["req"];
  res: CreateExpressContextOptions["res"];
  user: User | null;
};

export async function createContext(
  opts: CreateExpressContextOptions
): Promise<TrpcContext> {
  let user: User | null = null;

  try {
    user = await sdk.authenticateRequest(opts.req);
  } catch (error) {
    // Authentication is optional for public procedures.
    user = null;
  }

  return {
    req: opts.req,
    res: opts.res,
    user,
  };
}
import type { CookieOptions, Request } from "express";

export function getSessionCookieOptions(_req: Request): CookieOptions {
  // Preview is public HTTPS in a cross-site frame even when the upstream request is HTTP.
  return { httpOnly: true, path: "/", sameSite: "none", secure: true };
}
# Managed database

The managed database is MySQL-compatible and requires `features.server: true`. Capability declaration defaults and enablement rules are in [features](features.md). The platform supplies an empty database, not an ORM, application tables, migrations, or seed data.

## Connection and data scope

`DATABASE_URL` is the connection DSN in URL form and is a server secret. Managed development and the published container connect to the same database: development writes and schema changes affect the data used by the published application.

The development sandbox supplies the DSN to the shell and dev process. Its `DRIZZLE_DATABASE_URL` alias has the same value; the published deployment supplies `DATABASE_URL`, not that alias. [Local](../worklocally/SKILL.md) owns loading into a Local Host environment.

The database is project-scoped. The platform does not create an application-specific user, tenant schema, or row-ownership model.

## Persistence, migration and recovery

A deployment or process restart does not create a fresh managed database. Existing schema, data, and migration history survive those events. Provisioning does not execute application migrations.

Separate process startup from one-time data preparation. Use the application's migration history
or seed version to run only unfinished work; record completion only after that work succeeds.
A non-empty table alone does not prove initialization completed. Preserve subsequent user
edits rather than overwriting them with seed defaults on restart.

If startup depends on unfinished preparation, expose that state through application readiness
rather than guessing a fixed sleep. [Container startup](build-contracts.md#application-and-startup-preparation)
owns the platform health budget. Migration, seed, and database-check scripts are finite tasks:
release their own connections and temporary resources after success or failure, so completed
work does not leave a process running solely because its clients remain open.

This platform surface provides no separate development database, production-to-development copy, point-in-time recovery, or full asynchronous backup/export API.

## External database

With `features.database: false`, an application can supply an external `DATABASE_URL` through [protected secret configuration](secrets.md). The config plane rejects simultaneous ownership of `DATABASE_URL` by the managed database and a user-defined secret. The platform does not choose the external database's schema or lifecycle.

## Database implementation and change safety

Use the managed database by default for a server project's persistent application data. Choose `database: false` for a user-selected external database, or when the user explicitly chooses no database or a staged implementation without one; do not silently discard that choice. Explain the consequences when the requested behavior needs durable state that the selected design does not provide.

Read the DSN from the environment and use the TLS settings required by the selected driver. Keep migrations deterministic, repeatable, additive where possible, and committed with the application. Development and publication share the database, so treat development DDL and writes as changes to that same data.

Before destructive SQL, state what will be lost and obtain the user's approval for that destructive action. If a backup is required, use an available database-client or application export path; do not invent a platform restore endpoint. Check the affected application behavior after schema or migration changes.
import { eq } from "drizzle-orm";
import { drizzle } from "drizzle-orm/mysql2";
import { InsertUser, users } from "../drizzle/schema";
import { ENV } from './_core/env';

let _db: ReturnType<typeof drizzle> | null = null;

// Lazily create the drizzle instance so local tooling can run without a DB.
export async function getDb() {
  if (!_db && process.env.DATABASE_URL) {
    try {
      _db = drizzle(process.env.DATABASE_URL);
    } catch (error) {
      console.warn("[Database] Failed to connect:", error);
      _db = null;
    }
  }
  return _db;
}

export async function upsertUser(user: InsertUser): Promise<void> {
  if (!user.openId) {
    throw new Error("User openId is required for upsert");
  }

  const db = await getDb();
  if (!db) {
    throw new Error("Database is not available");
  }

  try {
    const values: InsertUser = {
      openId: user.openId,
    };
    const updateSet: Record<string, unknown> = {};

    const textFields = ["name", "email", "loginMethod"] as const;
    type TextField = (typeof textFields)[number];

    const assignNullable = (field: TextField) => {
      const value = user[field];
      if (value === undefined) return;
      const normalized = value ?? null;
      values[field] = normalized;
      updateSet[field] = normalized;
    };

    textFields.forEach(assignNullable);

    if (user.lastSignedIn !== undefined) {
      values.lastSignedIn = user.lastSignedIn;
      updateSet.lastSignedIn = user.lastSignedIn;
    }
    if (user.role !== undefined) {
      values.role = user.role;
      updateSet.role = user.role;
    } else if (user.openId === ENV.ownerOpenId) {
      values.role = 'admin';
      updateSet.role = 'admin';
    }

    if (!values.lastSignedIn) {
      values.lastSignedIn = new Date();
    }

    if (Object.keys(updateSet).length === 0) {
      updateSet.lastSignedIn = new Date();
    }

    await db.insert(users).values(values).onDuplicateKeyUpdate({
      set: updateSet,
    });
  } catch (error) {
    console.error("[Database] Failed to upsert user:", error);
    throw error;
  }
}

export async function getUserByOpenId(openId: string) {
  const db = await getDb();
  if (!db) {
    console.warn("[Database] Cannot get user: database not available");
    return undefined;
  }

  const result = await db.select().from(users).where(eq(users.openId, openId)).limit(1);

  return result.length > 0 ? result[0] : undefined;
}

// TODO: add feature queries here as your schema grows.
import { defineConfig } from "drizzle-kit";

const connectionString = process.env.DATABASE_URL;
if (!connectionString) {
  throw new Error("DATABASE_URL is required to run drizzle commands");
}

export default defineConfig({
  schema: "./drizzle/schema.ts",
  out: "./drizzle",
  dialect: "mysql",
  dbCredentials: {
    url: connectionString,
  },
});
import { Button } from "@/components/ui/button";
import { Loader2 } from "lucide-react";
import { Streamdown } from 'streamdown';

/**
 * All content in this page are only for example, replace with your own feature implementation
 * When building pages, remember your instructions in Frontend Workflow, Frontend Best Practices, Design Guide and Common Pitfalls
 */
export default function Home() {
  // If theme is switchable in App.tsx, we can implement theme toggling like this:
  // const { theme, toggleTheme } = useTheme();

  return (
    <div className="min-h-screen flex flex-col">
      <main>
        {/* Example: lucide-react for icons */}
        <Loader2 className="animate-spin" />
        Example Page
        {/* Example: Streamdown for markdown rendering */}
        <Streamdown>Any **markdown** content</Streamdown>
        <Button variant="default">Example Button</Button>
      </main>
    </div>
  );
}

