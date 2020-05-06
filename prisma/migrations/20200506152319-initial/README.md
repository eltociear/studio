# Migration `20200506152319-initial`

This migration has been generated by fmvilas at 5/6/2020, 3:23:19 PM.
You can check out the [state of the schema](./schema.prisma) after the migration.

## Database Steps

```sql
ALTER TABLE "public"."apis" DROP CONSTRAINT IF EXiSTS "apis_project_id_fkey";

ALTER TABLE "public"."organizations" ALTER COLUMN "feature_flags" DROP DEFAULT;

ALTER TABLE "public"."users" ALTER COLUMN "feature_flags" DROP DEFAULT;

ALTER TABLE "public"."apis" ADD FOREIGN KEY ("project_id")REFERENCES "public"."projects"("id") ON DELETE SET NULL  ON UPDATE CASCADE

ALTER INDEX "public"."invitations_uuid_unique" RENAME TO "invitations.uuid"

ALTER INDEX "public"."organizations_slug" RENAME TO "organizations.slug"

ALTER INDEX "public"."repos_url" RENAME TO "repos.url"

ALTER INDEX "public"."users_email" RENAME TO "users.email"

ALTER INDEX "public"."users_github_id" RENAME TO "users.github_id"

ALTER INDEX "public"."users_username" RENAME TO "users.username"
```

## Changes

```diff
diff --git schema.prisma schema.prisma
migration ..20200506152319-initial
--- datamodel.dml
+++ datamodel.dml
@@ -1,0 +1,103 @@
+generator client {
+  provider = "prisma-client-js"
+}
+
+datasource db {
+  provider = "postgresql"
+  url      = env("DATABASE_URL")
+}
+
+model apis {
+  asyncapi          String
+  computed_asyncapi Json
+  created_at        DateTime? @default(now())
+  creator_id        Int
+  id                Int       @default(autoincrement()) @id
+  link_path         String?
+  link_provider     String?
+  link_ref          String?
+  link_repo_id      Int?
+  project_id        Int?
+  title             String
+  version           String
+  users             users     @relation(fields: [creator_id], references: [id])
+  projects          projects? @relation(fields: [project_id], references: [id])
+}
+
+model invitations {
+  created_at      DateTime?     @default(now())
+  expires_at      DateTime
+  id              Int           @default(autoincrement())
+  inviter_id      Int
+  organization_id Int
+  role            String
+  scope           String?       @default("one")
+  uuid            String        @unique
+  users           users         @relation(fields: [inviter_id], references: [id])
+  organizations   organizations @relation(fields: [organization_id], references: [id])
+}
+
+model organizations {
+  created_at                  DateTime?             @default(now())
+  developer_portal_visibility String?               @default("hidden")
+  feature_flags               Json                  @default(dbgenerated())
+  id                          Int                   @default(autoincrement()) @id
+  name                        String
+  slug                        String                @unique
+  invitations                 invitations[]
+  organizations_users         organizations_users[]
+  projects                    projects[]
+  repos                       repos[]
+}
+
+model organizations_users {
+  created_at      DateTime?     @default(now())
+  organization_id Int
+  role            String        @default("member")
+  user_id         Int
+  organizations   organizations @relation(fields: [organization_id], references: [id])
+  users           users         @relation(fields: [user_id], references: [id])
+
+  @@unique([organization_id, user_id], name: "organizations_users_organization_id_user_id")
+}
+
+model projects {
+  created_at      DateTime?     @default(now())
+  creator_id      Int
+  id              Int           @default(autoincrement()) @id
+  name            String
+  organization_id Int
+  users           users         @relation(fields: [creator_id], references: [id])
+  organizations   organizations @relation(fields: [organization_id], references: [id])
+  apis            apis[]
+}
+
+model repos {
+  created_at      DateTime?     @default(now())
+  hook_id         Int?
+  id              Int           @default(autoincrement()) @id
+  organization_id Int
+  provider        String
+  provider_id     Int
+  title           String
+  url             String        @unique
+  organizations   organizations @relation(fields: [organization_id], references: [id])
+}
+
+model users {
+  avatar               String?
+  company              String?
+  created_at           DateTime?             @default(now())
+  display_name         String
+  email                String                @unique
+  feature_flags        Json                  @default(dbgenerated())
+  github_access_token  String?
+  github_id            String?               @unique
+  github_refresh_token String?
+  id                   Int                   @default(autoincrement()) @id
+  username             String                @unique
+  apis                 apis[]
+  invitations          invitations[]
+  organizations_users  organizations_users[]
+  projects             projects[]
+}
```

