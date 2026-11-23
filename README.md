# 口算练习 · 带数据库排行榜

纯静态页面（无构建步骤），成绩直接写入 Supabase，排行榜实时读库。

- 站点：index.html（单文件，原生 JS + fetch，无第三方依赖）
- 数据库：Supabase Postgres，表 `public.attempts`
- 部署：GitHub Pages（Actions 自动发布）

## 表结构

```sql
create table public.attempts (
  id bigserial primary key,
  nickname text not null check (char_length(nickname) between 1 and 20),
  correct int not null check (correct >= 0),
  total int not null check (total > 0 and correct <= total),
  seconds int check (seconds is null or seconds >= 0),
  created_at timestamptz not null default now()
);

alter table public.attempts enable row level security;

create policy "anyone can insert attempts"
  on public.attempts for insert to anon, authenticated with check (true);

create policy "anyone can read attempts"
  on public.attempts for select to anon, authenticated using (true);

grant select, insert on public.attempts to anon, authenticated;
grant usage, select on sequence public.attempts_id_seq to anon, authenticated;
```

RLS 只允许「插入自己的成绩」和「读取成绩」，不允许更新/删除；页面里放的是 anon/publishable key，属于公开密钥，安全性由 RLS 承担。

## 本地预览

```bash
python -m http.server 8080
# 打开 http://localhost:8080
```

## 部署

推送到 main 分支后，GitHub Actions（.github/workflows/pages.yml）自动发布到 GitHub Pages。
