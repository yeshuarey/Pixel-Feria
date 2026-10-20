# Pixel-Feria

## Configurar el contador global de fichas

La página estática usa Supabase para compartir el conteo entre visitantes.

1. Abre el [SQL Editor del proyecto](https://supabase.com/dashboard/project/uprhnwpsllrfdrosdjyd/sql/new).
2. Ejecuta este SQL para crear el contador y permitir consultar e incrementar únicamente su total:

```sql
create table public.fichas_counter (
  id integer primary key check (id = 1),
  total bigint not null default 0 check (total >= 0)
);

insert into public.fichas_counter (id, total)
values (1, 0)
on conflict (id) do nothing;

alter table public.fichas_counter enable row level security;

revoke all on table public.fichas_counter from anon, authenticated;
grant select on table public.fichas_counter to anon, authenticated;

create policy "Cualquiera puede consultar el total"
on public.fichas_counter
for select
to anon, authenticated
using (true);

create or replace function public.incrementar_fichas()
returns bigint
language sql
security definer
set search_path = ''
as $$
  update public.fichas_counter
  set total = total + 1
  where id = 1
  returning total;
$$;

revoke all on function public.incrementar_fichas() from public, anon, authenticated;
grant execute on function public.incrementar_fichas() to anon, authenticated;
```

3. La URL del proyecto y la clave pública `sb_publishable_...` ya están
   configuradas en `index.html`. Si necesitas revisarlas, abre
   [Project Settings → API Keys](https://supabase.com/dashboard/project/uprhnwpsllrfdrosdjyd/settings/api-keys).
   No pongas una clave `secret` o `service_role` en el código del navegador.
4. Publica de nuevo el sitio. El total se carga al abrir la página y aumenta al
   pulsar **Finalizar**. Si Supabase no está configurado o no responde, la ficha
   se sigue generando y se muestra el estado del contador.

El contador es público: como la página no requiere inicio de sesión, no puede
verificar que cada incremento corresponda a una persona distinta.

El total también se imprime en la tarjeta PNG. En un enlace QR abierto en otro
dispositivo, se usa el total cargado en ese dispositivo.
