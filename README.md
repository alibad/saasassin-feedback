# SaaSassin Feedback

**Free, open-source Canny alternative.** Keep product feedback, decisions, and customer communication in a portable open system.

Product feedback for product teams that need a public feedback loop without a proprietary archive.

- **Try it:** [feedback.saasassin.net](https://feedback.saasassin.net)
- **Research brief and product decision:** [saasassin.net/targets/canny](https://saasassin.net/targets/canny)

[![Deploy with Vercel](https://vercel.com/button)](https://vercel.com/new/clone?repository-url=https%3A%2F%2Fgithub.com%2Falibad%2Fsaasassin-feedback&project-name=saasassin-feedback&repository-name=saasassin-feedback&env=NEXT_PUBLIC_FIREBASE_API_KEY%2CNEXT_PUBLIC_FIREBASE_AUTH_DOMAIN%2CNEXT_PUBLIC_FIREBASE_PROJECT_ID%2CNEXT_PUBLIC_FIREBASE_APP_ID%2CFIREBASE_PROJECT_ID%2CFIREBASE_CLIENT_EMAIL%2CFIREBASE_PRIVATE_KEY&envDescription=Firebase+web+app+config+and+service-account+credentials+%28see+the+README%29&envLink=https%3A%2F%2Fgithub.com%2Falibad%2Fsaasassin-feedback%231-create-a-firebase-project)

Canny is a trademark of its owner and is named here only to describe what this product
replaces. SaaSassin Feedback is an independent, clean-room product and is not affiliated
with or endorsed by Canny.

## What you can do today

| Who | Can now |
|---|---|
| Team owner | Create a workspace, create public or private boards, share a board link |
| Anyone (signed out) | Read a public board at `/apps/feedback/b/<tenantId>/<boardSlug>`, search, filter by status/tag, sort by votes/newest/most discussed, open a post and read its discussion and staff response |
| Anyone (signed out) | Read the public portal at `/apps/feedback/b/<tenantId>`: roadmap columns (planned / started / completed), board list, and changelog |
| Signed-in Feedback user | Post an idea (auto-votes, like Fider), vote once per post and take the vote back, comment. Sign-in returns them to the board they came from |
| Owner / admin | Change status with a public note (decline needs a note), merge a duplicate into a canonical post (votes move, people who voted on both count once), tag posts, hide spam posts/comments with a recorded reason and restore them, see each post's voter list, publish changelog entries linked only to completed posts |
| Member | Read the workspace and triage with tags; viewer reads only |
| Poster | Sees similar existing ideas while typing a title, so they can vote instead of filing a duplicate |
| Owner / admin | Merge picker lists likely duplicates first; edit a post's wording (previous text kept in the audit log); add team-only internal notes (members can too) that never reach public pages or the Fider mapping |
| Anyone | Roadmap and changelog items deep-link straight to the post (`?post=<id>` opens it on the board) |
| Owner / admin | Unmerge a merged post: its previous status returns, moved votes come back (unless the voter removed them since), voters who had voted on both get their vote back; audited |
| Owner / admin / member | Assign a post owner from the team; filter the board by owner |
| Owner / admin | Record a voter's company and customer segment (per workspace) from the voter list; filter and rank posts by votes from a segment, see which companies want each post |
| Owner / admin | Group roadmap items into initiatives (themes) or releases (target dates); the public roadmap shows each with shipped progress |
| Owner / admin | Add signed outbound webhooks (HTTPS only, internal hosts refused) for `post.created` and `status.changed`, see the signing secret once, send a test, pause/resume, and read a delivery log with honest failures |
| Owner / admin | Invite teammates by email with a role, read the audit log of every mutation, download a full versioned JSON export or the Fider mapping alone, import a Canny posts CSV with a preview of mapping and problems before committing |

Full status, verification and known gaps: [docs/STATUS.md](docs/STATUS.md).

## Self-host

SaaSassin Feedback is a Next.js app backed by Firebase Authentication and Cloud Firestore.
You need Node.js 20.9 or newer, [pnpm](https://pnpm.io/installation) and a Firebase project of
your own.

### 1. Create a Firebase project

1. Create a project in the [Firebase console](https://console.firebase.google.com/).
2. **Firestore:** Build → Firestore Database → Create database.
3. **Authentication:** Build → Authentication → Get started. Feedback signs people in with
   **Google** and **Email/Password**; step 2 below enables both, or enable them yourself under
   Authentication → Sign-in method.
4. **Web app config:** Project settings → General → Your apps → add a Web app. Copy its config
   into the `NEXT_PUBLIC_FIREBASE_*` variables.
5. **Service account:** Project settings → Service accounts → Generate new private key. From the
   downloaded JSON, set `FIREBASE_PROJECT_ID` (`project_id`), `FIREBASE_CLIENT_EMAIL`
   (`client_email`) and `FIREBASE_PRIVATE_KEY` (`private_key`). Never commit the key, and never
   give these variables a `NEXT_PUBLIC_` prefix.
6. Once you deploy, add your domain under Authentication → Settings → Authorized domains
   (`localhost` is already allowed).

### 2. Deploy the rules, indexes and sign-in providers

```bash
pnpm install
pnpm exec firebase login
pnpm exec firebase deploy --only firestore:rules,firestore:indexes,auth --project <your-project-id>
```

`firestore.rules` denies all browser access to product data: every read and write goes through the server routes, which check the session, tenant membership and role with the Firebase Admin SDK. `auth` enables the Google and email/password providers listed in `firebase.json`.

### 3. Run it

```bash
cp .env.example .env.local   # fill in the values from step 1
pnpm dev
```

Open [http://localhost:3000](http://localhost:3000). This repository serves SaaSassin
Feedback at the root of every host: `next.config.ts` sets `STANDALONE_PRODUCT=feedback`,
and `src/proxy.ts` maps `/` to the product (`/apps/feedback`) and `/sign-in` to its sign-in page.

### 4. Deploy to Vercel

[![Deploy with Vercel](https://vercel.com/button)](https://vercel.com/new/clone?repository-url=https%3A%2F%2Fgithub.com%2Falibad%2Fsaasassin-feedback&project-name=saasassin-feedback&repository-name=saasassin-feedback&env=NEXT_PUBLIC_FIREBASE_API_KEY%2CNEXT_PUBLIC_FIREBASE_AUTH_DOMAIN%2CNEXT_PUBLIC_FIREBASE_PROJECT_ID%2CNEXT_PUBLIC_FIREBASE_APP_ID%2CFIREBASE_PROJECT_ID%2CFIREBASE_CLIENT_EMAIL%2CFIREBASE_PRIVATE_KEY&envDescription=Firebase+web+app+config+and+service-account+credentials+%28see+the+README%29&envLink=https%3A%2F%2Fgithub.com%2Falibad%2Fsaasassin-feedback%231-create-a-firebase-project)

The button copies this repository into your GitHub account, creates a Vercel project and asks for
the Firebase variables. To deploy an existing fork instead, import it in Vercel and add the
variables from `.env.example` under Project Settings → Environment Variables. Either way, add the
Vercel domain to your Firebase authorized domains afterwards.

## Tests

```bash
pnpm lint
pnpm typecheck
pnpm test        # unit tests
pnpm build
```

The Firestore suites run against the local Firestore emulator, which needs a Java 21+ JDK first on
your `PATH`:

```bash
export JAVA_HOME=/path/to/jdk-21   # macOS example: /Library/Java/JavaVirtualMachines/temurin-21.jdk/Contents/Home
export PATH="$JAVA_HOME/bin:$PATH"
pnpm test:rules      # Firestore security rules
pnpm test:emulator   # the golden workflow against a real Firestore emulator
```

## Contributing

Development happens in the SaaSassin monorepo,
[github.com/alibad/saasassin](https://github.com/alibad/saasassin), the single source of truth for
all ten SaaSassin products. This repository is generated from it by the monorepo's
`scripts/export-product.mjs`, so accepted changes land in the monorepo first and are then synced
here. Open issues and pull requests there or here.

Report vulnerabilities privately as described in [SECURITY.md](SECURITY.md).

## Talk to us

Moving off Canny, running Feedback for your team, or want to help build it?
[Tell us on saasassin.net](https://saasassin.net/targets/canny#interest).

## License

[GNU Affero General Public License v3.0](LICENSE) (AGPL-3.0-only). Use it, self-host it and modify
it; if you run a modified version as a network service, publish your changes under the same
license.
