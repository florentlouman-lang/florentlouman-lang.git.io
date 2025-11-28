[index.html](https://github.com/user-attachments/files/23829880/index.html)
<!DOCTYPE html>
<html lang="fr">
<head>
  <meta charset="UTF-8">
  <title>Lecteur Audio PWA iOS</title>
  <meta name="viewport" content="width=device-width, initial-scale=1.0">

  <!-- iOS PWA -->
  <meta name="apple-mobile-web-app-capable" content="yes">
  <meta name="apple-mobile-web-app-status-bar-style" content="black-translucent">
  <link rel="apple-touch-icon" href="icon-180.png">

  <!-- Manifest -->
  <link rel="manifest" href="manifest.json">
</head>
<body>
  <h1>Lecteur Audio PWA iOS</h1>
  <p>Application prête pour installation sur iPhone/iPad.</p>

  <script>
    if ('serviceWorker' in navigator) {
      navigator.serviceWorker.register('service-worker.js');
    }
  </script>
</body>
</html>
<img width="180" height="180" alt="icon-180" src="https://github.com/user-attachments/assets/d55122bb-bd9e-47ab-bd17-5b8848a66417" />
<img width="192" height="192" alt="icon-192" src="https://github.com/user-attachments/assets/5d96bb76-4875-45ac-91a5-8c3addb6e519" />
<img width="512" height="512" alt="icon-512" src="https://github.com/user-attachments/assets/7606be33-ba75-443f-ad29-d9b4a946a01d" />
[instructions_iOS.txt](https://github.com/user-attachments/files/23829889/instructions_iOS.txt)
Installation sur iOS :
1. Hébergez ce dossier sur un serveur HTTPS.
2. Ouvrez l’URL sur Safari (iPhone/iPad).
3. Appuyez sur le bouton "Partager".
4. Choisissez "Ajouter à l’écran d’accueil".
[manifest.json](https://github.com/user-attachments/files/23829894/manifest.json)
{
  "name": "Lecteur Audio",
  "short_name": "Audio",
  "start_url": "index.html",
  "display": "standalone",
  "background_color": "#101010",
  "theme_color": "#101010",
  "icons": [
    { "src": "icon-180.png", "sizes": "180x180", "type": "image/png" },
    { "src": "icon-192.png", "sizes": "192x192", "type": "image/png" },
    { "src": "icon-512.png", "sizes": "512x512", "type": "image/png" }
  ]
}
[service-worker.js](https://github.com/user-attachments/files/23829895/service-worker.js)
self.addEventListener('install', e => {
  e.waitUntil(
    caches.open('audio-cache').then(cache => {
      return cache.addAll(['index.html', 'manifest.json']);
    })
  );
});

self.addEventListener('fetch', e => {
  e.respondWith(
    caches.match(e.request).then(response => response || fetch(e.request))
  );
});
