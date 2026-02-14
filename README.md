# Hosting estático de juegos HTML en Home Assistant con Caddy (Fase 1)

## Estructura recomendada

```text
/share/webgames/site1/
  index.html
  assets/
  games/
    juego1/
      index.html
    juego2/
      index.html
```

En este repo se incluye una maqueta equivalente bajo `share/webgames/site1` para pruebas locales.

## Caddyfile (Fase 1)

- Archivo: `/share/caddy/Caddyfile`
- En este repo: `share/caddy/Caddyfile`

Características configuradas:
- `/g` sirve el sitio completo (`/srv/webgames/site1`)
- `/g/j1` y `/g/j2` disponibles (más alias `/j1` y `/j2`)
- sin directory listing
- solo métodos `GET`/`HEAD` (resto `405`)
- headers básicos de seguridad
- compresión `zstd` + `gzip`
- caché para assets estáticos

## Ejecución en contenedor/add-on Caddy

Montaje recomendado (solo lectura):
- host `/share/webgames` -> contenedor `/srv/webgames:ro`
- host `/share/caddy/Caddyfile` -> ruta de config del add-on/contenedor

Si tu add-on no soporta mapping personalizado, usa la ruta compartida disponible dentro del contenedor y ajusta `root` en Caddyfile.

## Actualizar juegos

1. Copia nueva versión sobre `/share/webgames/site1`.
2. Mantén rutas relativas de assets si quieres conservar `/g` como prefijo principal.
3. Recarga/reinicia el contenedor de Caddy si aplica.

## Exportación

- Backup simple: copia completa de `/share/webgames/site1`.
- O empaquetado:

```bash
zip -r site1-backup.zip /share/webgames/site1
```

## Pruebas rápidas (curl)

```bash
curl -I http://HA_HOST:8080/g
curl -I http://HA_HOST:8080/g/j1
curl -I http://HA_HOST:8080/j1
curl -X POST -I http://HA_HOST:8080/g
```

## Checklist de seguridad

- [x] No montar `/config` dentro del contenedor web.
- [x] Montar `/share/webgames` en read-only (si es posible).
- [x] Permitir solo `GET`/`HEAD`.
- [x] Añadir headers base (`nosniff`, `Referrer-Policy`, `X-Frame-Options`).
- [ ] Activar HSTS solo detrás de HTTPS real.
- [ ] Ajustar CSP por juego en fase posterior.

## Futuro (Fase 2, no implementado)

- `/api/*` -> `reverse_proxy api:3000`
- Mongo en red interna sin publicar puertos al host, con auth habilitada
