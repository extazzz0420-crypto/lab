# maveno-lab — витрина кейсов на lab.maveno.ru

Статический сайт для GitHub Pages. Всё бесплатно: репозиторий + Pages + сертификат.

## Что внутри

```
CNAME                 lab.maveno.ru          ← домен для GitHub Pages
index.html            витрина кейсов
assets/lab.css        стили витрины
assets/fonts/         Unbounded + Onest (self-hosted, variable woff2)
robots.txt
.nojekyll             отключает Jekyll-обработку
```

На витрину попадают только реальные проекты с согласия клиента. Тренировочные сборки
и демо лежат вне репозитория (`~/sites/_standby-*`) и не публикуются.

## Деплой (один раз)

```bash
cd ~/sites/maveno-lab
git init -b main
git add -A && git commit -m "lab: витрина кейсов"
gh repo create extazzz0420-crypto/lab --public --source=. --remote=origin --push
# включить Pages: ветка main, корень
gh api -X POST repos/extazzz0420-crypto/lab/pages -f 'source[branch]=main' -f 'source[path]=/' 
# привязать домен (CNAME в репозитории GitHub подхватит сам, но фиксируем явно)
gh api -X PUT repos/extazzz0420-crypto/lab/pages -f cname=lab.maveno.ru -F https_enforced=true
```

Сертификат Let's Encrypt GitHub выпускает сам после того, как DNS начнёт резолвиться —
обычно 5–20 минут, иногда до часа. До этого https отдаёт ошибку, http работает.

## DNS (делается в панели Cloudflare — один раз)

Домен maveno.ru делегирован на Cloudflare (`fay.ns.cloudflare.com`, `ruben.ns.cloudflare.com`),
поэтому записи правятся только в панели Cloudflare, не у регистратора.

**DNS → Records → Add record:**

| Поле | Значение |
|---|---|
| Type | `CNAME` |
| Name | `lab` |
| Target | `extazzz0420-crypto.github.io` |
| Proxy status | **DNS only** (серое облако, не оранжевое) |
| TTL | Auto |

Прокси Cloudflare для Pages выключаем намеренно: с оранжевым облаком GitHub не может
выпустить сертификат и подтвердить домен, а кэш Cloudflare начинает отдавать старые версии.

Записи `maveno.ru` (A → 185.199.108–111.153) и `www` (CNAME → extazzz0420-crypto.github.io)
**не трогаем** — это основной сайт.

## Добавление нового лендинга

```bash
cp -r <новая-папка-проекта> ~/sites/maveno-lab/<slug>/
# карточку в index.html (секция .list), счётчик «00 — 00» и «Опубликовано N кейсов»
git add -A && git commit -m "lab: + <slug>" && git push
```

Публиковать только после письменного «ок» клиента на показ проекта в портфолио.

Ссылка для клиента: `https://lab.maveno.ru/<slug>/`

Требование к лендингу: только относительные пути (`styles/main.css`, `fonts/...`),
никаких `/styles/...` от корня — иначе в подпапке отвалится.

## Проверка после DNS

```bash
dig +short lab.maveno.ru                      # должен вернуть 185.199.108-111.153
curl -sI https://lab.maveno.ru/ | head -3      # 200 и заголовок server: GitHub.com
```
