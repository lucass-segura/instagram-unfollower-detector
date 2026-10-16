# Instagram Unfollower Detector

Este script te permite detectar quién no te sigue de vuelta en Instagram. Simplemente sigue los pasos a continuación para utilizarlo.


## Instrucciones

1. **Ve a tu perfil de Instagram.**
   
3. **Abre la Consola del Navegador:**
   - Presiona `F12` o haz clic derecho en cualquier parte de la página y selecciona "Inspeccionar" o "Inspeccionar elemento", luego ve a la pestaña "Consola".
     
4. **Pega el Código en la Consola:**
   - Copia todo el código proporcionado a continuación y pégalo en la consola del navegador.
     ```
     (async () => {
  const getCookie = (n) =>
    document.cookie.split('; ').find((c) => c.startsWith(n + '='))?.split('=')[1];
  const sleep = (ms) => new Promise((r) => setTimeout(r, ms));
  const esc = (s) =>
    String(s ?? '').replace(/[&<>"']/g, (c) => ({ '&': '&amp;', '<': '&lt;', '>': '&gt;', '"': '&quot;', "'": '&#39;' }[c]));
  const fmt = (n) => Number(n).toLocaleString('es-AR');

  const headers = {
    'X-IG-App-ID': '936619743392459',
    'X-CSRFToken': getCookie('csrftoken') || '',
    'X-Requested-With': 'XMLHttpRequest',
  };
  const get = (url) => fetch(url, { headers, credentials: 'include' });

  // ================== INTERFAZ ==================
  // Trusted Types por si Instagram lo exige
  let policy = null;
  try { policy = window.trustedTypes?.createPolicy('igu-' + Date.now(), { createHTML: (s) => s }); } catch {}
  const html = (s) => (policy ? policy.createHTML(s) : s);

  document.getElementById('igu-host')?.remove();
  const host = document.createElement('div');
  host.id = 'igu-host';
  document.body.appendChild(host);
  const shadow = host.attachShadow({ mode: 'open' });

  const css = `
  :host{all:initial}
  *{box-sizing:border-box;font-family:-apple-system,BlinkMacSystemFont,"Segoe UI",Roboto,Helvetica,Arial,sans-serif}
  .wrap{position:fixed;inset:0;z-index:2147483647;display:grid;place-items:center;background:rgba(8,8,12,.55);backdrop-filter:blur(6px);animation:fade .3s ease}
  .card{width:min(420px,92vw);background:#16161d;color:#f5f5f7;border-radius:22px;padding:22px;position:relative;overflow:hidden;
    box-shadow:0 24px 70px rgba(0,0,0,.55),0 0 0 1px rgba(255,255,255,.06);animation:up .5s cubic-bezier(.2,.9,.3,1.2)}
  .blob{position:absolute;width:260px;height:260px;border-radius:50%;filter:blur(40px);opacity:.35;pointer-events:none}
  .b1{top:-120px;right:-80px;background:#d62976;animation:float 7s ease-in-out infinite alternate}
  .b2{bottom:-140px;left:-100px;background:#4f5bd5;animation:float 9s ease-in-out infinite alternate-reverse}
  .content{position:relative}
  .head{display:flex;gap:14px;align-items:center;margin-bottom:18px}
  .ring{width:58px;height:58px;flex:none;border-radius:50%;display:grid;place-items:center;
    background:conic-gradient(from 0deg,#feda75,#fa7e1e,#d62976,#962fbf,#4f5bd5,#feda75);animation:spin 1.4s linear infinite;transition:background .4s}
  .inner{width:50px;height:50px;border-radius:50%;background:#16161d;display:grid;place-items:center;font-size:24px;animation:spin 1.4s linear infinite reverse}
  .ring.done{animation:pop .5s ease;background:linear-gradient(135deg,#4ade80,#16a34a)}
  .ring.error{animation:none;background:#ef4444}
  .ring.done .inner,.ring.error .inner{animation:none}
  .title{font-size:18px;font-weight:700}
  .sub{font-size:13px;color:#a1a1aa;margin-top:3px;min-height:16px}
  .row{margin:14px 0;transition:opacity .3s}
  .row.idle{opacity:.4}
  .label{display:flex;justify-content:space-between;font-size:13px;margin-bottom:7px;color:#d4d4d8}
  .count{font-variant-numeric:tabular-nums;color:#a1a1aa}
  .bar{height:10px;border-radius:99px;background:#26262f;overflow:hidden}
  .fill{height:100%;width:0;border-radius:99px;position:relative;overflow:hidden;transition:width .6s cubic-bezier(.3,.8,.3,1);
    background:linear-gradient(90deg,#feda75,#fa7e1e,#d62976,#962fbf,#4f5bd5,#feda75);background-size:200% 100%;animation:shift 2s linear infinite}
  .fill::after{content:"";position:absolute;inset:0;background:linear-gradient(90deg,transparent,rgba(255,255,255,.5),transparent);transform:translateX(-100%);animation:shine 1.6s ease-in-out infinite}
  .fill.indet{width:35%!important;animation:shift 2s linear infinite,indet 1.4s ease-in-out infinite}
  .row.done .fill{width:100%!important;animation:none;transform:none}
  .row.done .fill::after{animation:none}
  .pause{display:flex;align-items:center;gap:12px;border-radius:14px;background:#1f1f29;font-size:13px;
    max-height:0;opacity:0;padding:0 12px;margin-top:0;overflow:hidden;transition:all .35s ease}
  .pause.on{max-height:90px;opacity:1;padding:12px;margin-top:14px}
  .clock{position:relative;width:44px;height:44px;flex:none}
  .clock svg{transform:rotate(-90deg)}
  .clock circle{fill:none;stroke-width:4}
  .clock .bg{stroke:#2e2e3a}
  .clock .fg{stroke:url(#igu-g);stroke-linecap:round;stroke-dasharray:113.1;stroke-dashoffset:0}
  .pause.on .fg{animation:drain var(--d,10s) linear forwards}
  .secs{position:absolute;inset:0;display:grid;place-items:center;font-weight:700;font-size:14px;font-variant-numeric:tabular-nums}
  .ptitle{font-weight:600}
  .psub{color:#a1a1aa;font-size:12px;margin-top:2px}
  .results{display:none;margin-top:6px}
  .results.on{display:block;animation:up .4s ease}
  .summary{font-size:14px;margin-bottom:10px;color:#d4d4d8}
  .summary b{background:linear-gradient(90deg,#fa7e1e,#d62976,#962fbf);-webkit-background-clip:text;background-clip:text;color:transparent;font-size:16px}
  .list{max-height:300px;overflow-y:auto;margin:0 -6px;padding:0 6px}
  .list::-webkit-scrollbar{width:6px}.list::-webkit-scrollbar-thumb{background:#33333f;border-radius:9px}
  .item{display:flex;align-items:center;gap:10px;padding:8px;border-radius:12px;text-decoration:none;color:inherit;opacity:0;animation:up .4s ease forwards;transition:background .2s}
  .item:hover{background:#22222c}
  .u{font-size:14px;font-weight:600}
  .n{font-size:12px;color:#a1a1aa;white-space:nowrap;overflow:hidden;text-overflow:ellipsis;max-width:260px}
  .v{color:#3b82f6;font-size:12px;margin-left:4px}
  .empty{text-align:center;padding:20px;font-size:15px}
  .actions{display:flex;gap:8px;margin-top:14px}
  button{flex:1;border:0;border-radius:12px;padding:11px;font-weight:600;font-size:13px;cursor:pointer;color:#fff;background:#26262f;transition:transform .15s,filter .15s}
  button:hover{transform:translateY(-1px);filter:brightness(1.15)}
  button.primary{background:linear-gradient(90deg,#fa7e1e,#d62976,#962fbf)}
  @keyframes spin{to{transform:rotate(360deg)}}
  @keyframes shift{to{background-position:-200% 0}}
  @keyframes shine{60%,100%{transform:translateX(100%)}}
  @keyframes indet{0%{transform:translateX(-100%)}100%{transform:translateX(300%)}}
  @keyframes drain{to{stroke-dashoffset:113.1}}
  @keyframes up{from{opacity:0;transform:translateY(12px)}to{opacity:1;transform:none}}
  @keyframes fade{from{opacity:0}}
  @keyframes float{to{transform:translate(-50px,40px) scale(1.15)}}
  @keyframes pop{0%{transform:scale(.8)}60%{transform:scale(1.12)}100%{transform:scale(1)}}
  `;
  const sheet = new CSSStyleSheet();
  sheet.replaceSync(css);
  shadow.adoptedStyleSheets = [sheet];

  shadow.innerHTML = html(`
  <div class="wrap">
    <div class="card">
      <div class="blob b1"></div><div class="blob b2"></div>
      <div class="content">
        <div class="head">
          <div class="ring" id="ring"><div class="inner" id="emoji">👀</div></div>
          <div>
            <div class="title">¿Quién no te sigue?</div>
            <div class="sub" id="status">Preparando todo...</div>
          </div>
        </div>
        <div id="progress">
          <div class="row idle" data-k="following">
            <div class="label"><span>Seguidos</span><span class="count">—</span></div>
            <div class="bar"><div class="fill"></div></div>
          </div>
          <div class="row idle" data-k="followers">
            <div class="label"><span>Seguidores</span><span class="count">—</span></div>
            <div class="bar"><div class="fill"></div></div>
          </div>
        </div>
        <div class="pause" id="pause">
          <div class="clock">
            <svg width="44" height="44" viewBox="0 0 44 44">
              <defs><linearGradient id="igu-g" x1="0" y1="0" x2="1" y2="1">
                <stop offset="0" stop-color="#fa7e1e"/><stop offset=".5" stop-color="#d62976"/><stop offset="1" stop-color="#4f5bd5"/>
              </linearGradient></defs>
              <circle class="bg" cx="22" cy="22" r="18"/><circle class="fg" cx="22" cy="22" r="18"/>
            </svg>
            <div class="secs" id="secs">10</div>
          </div>
          <div><div class="ptitle" id="ptitle">Pausa anti-bloqueo</div><div class="psub" id="psub"></div></div>
        </div>
        <div class="results" id="results"></div>
      </div>
    </div>
  </div>`);

  const $ = (s) => shadow.querySelector(s);
  const row = (k) => shadow.querySelector(`.row[data-k="${k}"]`);
  const setStatus = (t) => ($('#status').textContent = t);

  function setProgress(k, n, total) {
    const r = row(k);
    const f = r.querySelector('.fill');
    r.classList.remove('idle');
    r.querySelector('.count').textContent = total ? `${fmt(n)} / ${fmt(total)}` : fmt(n);
    if (total) {
      f.classList.remove('indet');
      f.style.width = Math.min(100, (n / total) * 100) + '%';
    } else {
      f.classList.add('indet');
    }
  }

  const frases = [
    'Tomate un tecito de manzanilla 🍵',
    'Estirá las piernas un toque 🧘',
    'Instagram respira, vos también 😮‍💨',
    'Paciencia, que no te bloqueen 🛡️',
  ];

  async function pausa(seg, titulo, sub) {
    const p = $('#pause');
    $('#ptitle').textContent = titulo;
    $('#psub').textContent = sub || frases[Math.floor(Math.random() * frases.length)];
    p.style.setProperty('--d', seg + 's');
    p.classList.remove('on');
    void p.offsetWidth; // reinicia la animación
    p.classList.add('on');
    for (let s = seg; s > 0; s--) {
      $('#secs').textContent = s;
      await sleep(1000);
    }
    p.classList.remove('on');
  }

  function fallo(msg) {
    $('#ring').classList.add('error');
    $('#emoji').textContent = '⚠️';
    setStatus(msg);
    const res = $('#results');
    res.classList.add('on');
    res.innerHTML = html(`<div class="actions"><button id="close">Cerrar</button></div>`);
    $('#close').onclick = () => host.remove();
  }

  // ================== LÓGICA ==================
  let userId = getCookie('ds_user_id');
  if (!userId) {
    const username = prompt('No encontré tu ID. Escribí tu usuario de Instagram:');
    try {
      const r = await get(`/api/v1/users/web_profile_info/?username=${encodeURIComponent(username)}`);
      userId = (await r.json())?.data?.user?.id;
    } catch {}
  }
  if (!userId) return fallo('No pude obtener tu ID. ¿Estás logueado?');

  // Totales para las barras de progreso
  const totales = {};
  try {
    const u = (await (await get(`/api/v1/users/${userId}/info/`)).json()).user;
    totales.following = u.following_count;
    totales.followers = u.follower_count;
  } catch {}

  async function fetchList(tipo) {
    const lista = new Map();
    let maxId = '';
    let pagina = 0;
    let reintentos = 0;
    setStatus(tipo === 'following' ? 'Revisando a quién seguís...' : 'Revisando quién te sigue...');
    setProgress(tipo, 0, totales[tipo]);

    do {
      const url =
        `/api/v1/friendships/${userId}/${tipo}/?count=50` +
        (maxId ? `&max_id=${maxId}` : '') +
        (tipo === 'followers' ? '&search_surface=follow_list_page' : '');

      let res;
      try { res = await get(url); } catch { res = null; }

      if (!res || res.status === 429 || res.status >= 500) {
        if (++reintentos > 5) throw new Error(`Demasiados errores trayendo ${tipo}`);
        await pausa(60, `Instagram pidió un respiro (${reintentos}/5)`, 'Espero un minuto y sigo solo');
        continue;
      }
      if (!res.ok) throw new Error(`HTTP ${res.status} trayendo ${tipo}`);

      reintentos = 0;
      const data = await res.json();
      (data.users || []).forEach((u) => lista.set(String(u.pk ?? u.pk_id ?? u.id), u));
      maxId = data.next_max_id || '';
      pagina++;
      setProgress(tipo, lista.size, totales[tipo]);

      if (!maxId) break;
      await sleep(800 + Math.random() * 400);
      if (pagina % 10 === 0) await pausa(10, 'Pausa anti-bloqueo');
    } while (maxId);

    row(tipo).classList.add('done');
    row(tipo).querySelector('.count').textContent = fmt(lista.size);
    return lista;
  }

  try {
    const seguidos = await fetchList('following');
    const seguidores = await fetchList('followers');

    const noTeSiguen = [...seguidos.entries()]
      .filter(([id]) => !seguidores.has(id))
      .map(([id, u]) => ({
        id,
        username: u.username,
        nombre: u.full_name,
        verificado: !!u.is_verified,
        privado: !!u.is_private,
        link: `https://www.instagram.com/${u.username}/`,
      }));

    const json = JSON.stringify(noTeSiguen, null, 2);
    const nombreArchivo = 'usersNotFollowingBack.json';

    const descargar = async (btn) => {
      const original = btn.textContent;
      const ok = () => {
        btn.textContent = '¡Descargado!';
        setTimeout(() => (btn.textContent = original), 1800);
      };

      // Opción 1: diálogo nativo de "Guardar como" (Chrome / Edge)
      if (window.showSaveFilePicker) {
        try {
          const handle = await window.showSaveFilePicker({
            suggestedName: nombreArchivo,
            types: [{ description: 'JSON', accept: { 'application/json': ['.json'] } }],
          });
          const w = await handle.createWritable();
          await w.write(json);
          await w.close();
          return ok();
        } catch (e) {
          if (e.name === 'AbortError') return; // cancelaste el diálogo
        }
      }

      // Opción 2: link de descarga clásico, agregado al DOM
      try {
        const url = URL.createObjectURL(new Blob([json], { type: 'application/json' }));
        const a = document.createElement('a');
        a.href = url;
        a.download = nombreArchivo;
        a.style.display = 'none';
        shadow.appendChild(a);
        a.click();
        a.remove();
        setTimeout(() => URL.revokeObjectURL(url), 5000);
        return ok();
      } catch {}

      // Opción 3: si todo falla, copia el JSON al portapapeles
      await navigator.clipboard.writeText(json);
      btn.textContent = 'JSON copiado';
      setTimeout(() => (btn.textContent = original), 1800);
    };

    // Resultado
    $('#ring').classList.add('done');
    $('#emoji').textContent = noTeSiguen.length ? '🐀' : '🎉';
    setStatus(`Seguís a ${fmt(seguidos.size)} · Te siguen ${fmt(seguidores.size)}`);

    const items = noTeSiguen
      .map((u, i) => `
        <a class="item" href="${esc(u.link)}" target="_blank" rel="noopener" style="animation-delay:${Math.min(i * 35, 800)}ms">
          <div>
            <div class="u">@${esc(u.username)}${u.verificado ? '<span class="v">✔</span>' : ''}</div>
            <div class="n">${esc(u.nombre)}</div>
          </div>
        </a>`)
      .join('');

    const res = $('#results');
    res.classList.add('on');
    res.innerHTML = html(
      (noTeSiguen.length
        ? `<div class="summary"><b>${fmt(noTeSiguen.length)}</b> ratas no te siguen de vuelta</div><div class="list">${items}</div>`
        : `<div class="empty">Todos te siguen de vuelta 🎉</div>`) +
      `<div class="actions">
        ${noTeSiguen.length ? '<button class="primary" id="dl">Descargar JSON</button><button id="copy">Copiar usuarios</button>' : ''}
        <button id="close">Cerrar</button>
      </div>`
    );

    $('#close').onclick = () => host.remove();
    if (noTeSiguen.length) {
      $('#dl').onclick = (e) => descargar(e.currentTarget);
      $('#copy').onclick = async (e) => {
        await navigator.clipboard.writeText(noTeSiguen.map((u) => '@' + u.username).join('\n'));
        e.target.textContent = '¡Copiado!';
        setTimeout(() => (e.target.textContent = 'Copiar usuarios'), 1500);
      };
    }

    console.table(noTeSiguen.map(({ username, nombre, verificado }) => ({ username, nombre, verificado })));
  } catch (e) {
    fallo(`Error: ${e.message}`);
    console.error(e);
  }
})();
       ```
     
5. **Presiona Enter para Ejecutar el Script:**
   - Una vez que hayas pegado el código, presiona `Enter` para iniciar el script.
     
6. **Espera a que el Script Termine:**
   - El script recorrerá tus seguidores y mostrará aquellos que no te siguen de vuelta. Ten paciencia mientras se ejecuta.


## Código Explicado

El código proporcionado es un script JavaScript que utiliza la API de Instagram para obtener una lista de tus seguidores y determinar quién no te sigue de vuelta. Aquí hay una breve explicación de algunas partes clave del código:

- **Función `getCookie(b)`:** Esta función se utiliza para obtener el valor de una cookie específica del navegador.
- **Función `sleep(a)`:** Esta función devuelve una promesa que se resuelve después de un tiempo determinado, lo que permite pausar la ejecución del script.
- **Funciones `afterUrlGenerator(a)` y `unfollowUserUrlGenerator(a)`:** Estas funciones generan las URL necesarias para obtener más seguidores y para dejar de seguir a un usuario, respectivamente.
- **Variables `csrftoken` y `ds_user_id`:** Estas variables almacenan el token CSRF y el ID de usuario necesarios para realizar las solicitudes a la API de Instagram.
- **Función `startScript()`:** Esta función inicia el script, realiza solicitudes a la API de Instagram, recorre los seguidores y muestra aquellos que no te siguen de vuelta en la consola del navegador.

Una vez que el script haya terminado de ejecutarse, se descargará un archivo JSON con la lista de usuarios que no te siguen de vuelta.
