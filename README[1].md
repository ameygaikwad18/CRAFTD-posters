# CRAFTD website

This is a Vercel-ready poster shop. The public site is in `index.html`; Supabase provides the owner login, catalogue, showcase, and image storage.

## Deploy with GitHub and Vercel

1. Create a new GitHub repository, for example `craftd-studio`.
2. Upload the complete contents of this folder — including `api/`, `package.json`, and `index.html` — to the repository root.
3. Import the GitHub repository in Vercel. Vercel deploys the site and API routes on every push to `main`.
4. Set up Supabase using the instructions below, then redeploy from Vercel.

## Before launch

- Customer ordering is WhatsApp-first. Ready-made checkout and custom-poster requests open a pre-filled WhatsApp chat to `+91 8989251028`; customers attach their reference images and payment screenshots directly inside WhatsApp.
- Create a Supabase owner user with your email and a strong password. Use those details in **Owner login** on the deployed site.
- Click **Owner login** to turn editing on. You can then use **Add poster** for the ready-made shop and **Add showcase image** in “Made to get noticed.” Every poster and showcase tile gains a **Delete** button while owner editing is on.
- GitHub stores your code; Supabase stores poster art, catalogue data, and showcase entries. Your owner edits survive new Vercel deployments.
- Replace the demo portfolio names and placeholder copy with real campus projects.
- No FormSubmit setup is needed. FormSubmit can be removed from your browser tabs.
- `qr-payment.png` is already included and cropped to the scannable QR only. Keep this filename and file at the repository root when uploading to GitHub. Customers must upload a payment screenshot plus their delivery address before the order request reaches your email.

## Supabase setup

1. In Supabase, create a user under **Authentication → Users → Add user** using your owner email and password.
2. Open **SQL Editor**, create a new query, and run this:

```sql
create table if not exists public.posters (
  id text primary key,
  name text not null,
  category text not null,
  price text not null,
  edition text not null,
  image text,
  created_at timestamptz default now()
);

create table if not exists public.showcase (
  id text primary key,
  title text not null,
  type text not null,
  image text,
  created_at timestamptz default now()
);

alter table public.posters enable row level security;
alter table public.showcase enable row level security;

create policy "Anyone can view posters" on public.posters for select using (true);
create policy "Owner manages posters" on public.posters for all to authenticated using ((auth.jwt() ->> 'email') = 'craftdstudio26@gmail.com') with check ((auth.jwt() ->> 'email') = 'craftdstudio26@gmail.com');
create policy "Anyone can view showcase" on public.showcase for select using (true);
create policy "Owner manages showcase" on public.showcase for all to authenticated using ((auth.jwt() ->> 'email') = 'craftdstudio26@gmail.com') with check ((auth.jwt() ->> 'email') = 'craftdstudio26@gmail.com');

insert into storage.buckets (id, name, public) values ('craftd-images', 'craftd-images', true) on conflict (id) do nothing;
create policy "Anyone can view poster images" on storage.objects for select using (bucket_id = 'craftd-images');
create policy "Owner uploads poster images" on storage.objects for insert to authenticated with check (bucket_id = 'craftd-images' and (auth.jwt() ->> 'email') = 'craftdstudio26@gmail.com');
```

For a single-owner store, these authenticated policies are fine. If you add team members later, restrict the write policies to approved user IDs.
