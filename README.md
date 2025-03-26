# 🚀 Projet : Monitoring Serveur Jeu (Node.js/Socket.IO)

## 🎯 Objectif du Projet

Créer une interface web sécurisée permettant de :

- ✅ Voir les statistiques du serveur (CPU, RAM, uptime)
- ✅ Afficher en temps réel les joueurs connectés
- ✅ Gérer facilement le serveur de jeu (démarrer, arrêter, redémarrer)
- ✅ Protéger l'accès via authentification par mot de passe

---

## 🛠️ Technologies utilisées

- **Serveur :** Node.js, Express.js, Socket.IO
- **Hébergement :** VPS Ubuntu (Linode)
- **Process manager :** PM2
- **Proxy et SSL :** Apache2, Let's Encrypt
- **Frontend :** HTML, JavaScript, TailwindCSS

---

## 🗂️ Structure du projet

```
/home/gauthier/
├── monitoring/             # Serveur Monitoring
│   ├── app.js              # API Node.js Monitoring
│   ├── index.html          # Interface web Monitoring
│   ├── package.json
│   └── node_modules
│
├── Projet websocket unity nodejs 2/  # Serveur Jeu
│   ├── index.js            # API Socket.IO (serveur jeu)
│   ├── game.js             # Gestion logique jeu
│   ├── chat.js             # Gestion du chat
│   ├── store.js            # Liste joueurs connectés
│   └── node_modules
```

---

## 📚 Étapes réalisées

### 1. Serveur Jeu Node.js (port 3000)

- Installation Express, Socket.IO, CORS
- Implémentation de Socket.IO (gestion joueurs)
- Ajout route API `/players` pour joueurs connectés

```js
app.get("/players", (req, res) => {
  const playersArray = Object.entries(store.globalPlayers).map(([id, pseudo]) => ({ id, pseudo }));
  res.json({ players: playersArray });
});
```

---

### 2. Serveur Monitoring Node.js (port 8080)

- API Node.js affichant CPU, RAM, uptime via PM2
- Routes `/stats` et `/action` pour statistiques et gestion serveur

```js
app.get('/stats', async (req, res) => {
  exec('pm2 jlist', (error, stdout) => {
    const pm2List = JSON.parse(stdout);
    const serverJeu = pm2List.find(p => p.name === 'serveur-jeu');
    const isRunning = serverJeu && serverJeu.pm2_env.status === 'online';

    res.json({
      cpu: isRunning ? serverJeu.monit.cpu.toFixed(2) : '0.00',
      ram: isRunning ? (serverJeu.monit.memory / (1024 * 1024)).toFixed(2) : '0.00',
      uptime: isRunning ? Date.now() - serverJeu.pm2_env.pm_uptime : null,
      serverRunning: isRunning
    });
  });
});
```

---

### 3. Interface web du Monitoring

- Frontend avec TailwindCSS
- Actualisation automatique via JavaScript (toutes les 5 sec)
- Affichage stats et liste joueurs connectés en temps réel

---

### 4. Hébergement Apache & HTTPS (Let's Encrypt)

- Création sous-domaine `monitoring.gauthierpl.lol`
- Certificat SSL avec Let's Encrypt

```bash
sudo certbot --apache -d monitoring.gauthierpl.lol
```

- Configuration reverse-proxy Apache

```apache
<VirtualHost *:443>
  ServerName monitoring.gauthierpl.lol

  ProxyPreserveHost On
  ProxyRequests Off

  ProxyPass /stats http://localhost:8080/stats
  ProxyPassReverse /stats http://localhost:8080/stats

  ProxyPass /action http://localhost:8080/action
  ProxyPassReverse /action http://localhost:8080/action

  ProxyPass /players http://localhost:3000/players
  ProxyPassReverse /players http://localhost:3000/players

  ProxyPass / http://localhost:8080/
  ProxyPassReverse / http://localhost:8080/

  SSLEngine On
  SSLCertificateFile /etc/letsencrypt/live/monitoring.gauthierpl.lol/fullchain.pem
  SSLCertificateKeyFile /etc/letsencrypt/live/monitoring.gauthierpl.lol/privkey.pem
</VirtualHost>
```

---

### 5. Protection par mot de passe (Apache)

```bash
sudo htpasswd -c /etc/apache2/.htpasswd gauthier
```

```apache
<Location />
    AuthType Basic
    AuthName "Accès protégé"
    AuthUserFile /etc/apache2/.htpasswd
    Require valid-user
</Location>
```

---

## 🚦 Commandes utiles

```bash
# PM2 (Gestion serveur Node.js)
pm2 start index.js --name serveur-jeu
pm2 start app.js --name monitoring
pm2 restart serveur-jeu
pm2 restart monitoring

# Recharger Apache
sudo systemctl reload apache2
```

---

## 🔗 Résultat final

- 🌐 [monitoring.gauthierpl.lol](https://monitoring.gauthierpl.lol)
- 🔒 Authentification par mot de passe
- 📊 Stats serveur dynamiques
- 👥 Liste joueurs actualisée en temps réel

---

## ✨ Conclusion

Projet fonctionnel, sécurisé et accessible 24/7 en HTTPS, facilitant la maintenance et la gestion du serveur de jeu.
