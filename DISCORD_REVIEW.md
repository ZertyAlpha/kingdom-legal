# Revue Discord — 19 octobre 2026

## Résultat et périmètre

**La conformité complète n'est pas établie. Ne pas publier la PR comme une certification de conformité.**

Revue statique ciblée de ZertyAlpha/RPG, branche main au commit `d6928e14da779af67a1d66b1766e68dded7e050b`, et comparaison avec les textes FR/EN de cette branche. Les recherches GitHub portent sur cette version de main ; une absence de résultat n'est pas une preuve d'absence dans tout le dépôt. Le code de production, la configuration effective, les disques, le proxy et le Developer Portal ne sont pas accessibles ici. Aucun test en production ni effacement réel n'a été effectué. Le bot n'a pas été modifié.

Sources Discord consultées :
- https://support-dev.discord.com/hc/articles/8562894815383-Discord-Developer-Terms-of-Service
- https://support-dev.discord.com/hc/articles/8563934450327-Discord-Developer-Policy

## Constats prioritaires

| Sujet | Preuve dans le code | Conclusion / action |
|---|---|---|
| Suppression de personnage | [src/services/AdminResetService.js](https://github.com/ZertyAlpha/RPG/blob/d6928e14da779af67a1d66b1766e68dded7e050b/src/services/AdminResetService.js) : resetPlayer conserve users et les historiques ; deleteCharacter copie les données dans character_backups ; restoreCharacter les réécrit | Ce mécanisme de modération ne traite pas une demande d'effacement. Prévoir une procédure distincte couvrant les identifiants, historiques, notifications, copies et sauvegardes ; documenter toute conservation légalement requise. Aucun parcours complet d'effacement n'a été identifié dans la revue. |
| Historiques persistants | [src/database/schema.sql](https://github.com/ZertyAlpha/RPG/blob/d6928e14da779af67a1d66b1766e68dded7e050b/src/database/schema.sql) : house_storage_transactions explicitement conservé après reset ; character_backups contient user_id, nom, auteur, JSON, dates | Définir une conservation justifiée et un effacement/anonymisation des références personnelles. Aucun effacement automatique des copies de personnage n'a été identifié. |
| Sauvegardes | [src/database/backup.js](https://github.com/ZertyAlpha/RPG/blob/d6928e14da779af67a1d66b1766e68dded7e050b/src/database/backup.js) : VACUUM INTO puis rotation par nombre et préfixe ; [src/config/config.js](https://github.com/ZertyAlpha/RPG/blob/d6928e14da779af67a1d66b1766e68dded7e050b/src/config/config.js) : BACKUP_KEEP=20 par défaut, 0=aucun nettoyage | Pas de durée maximale dans ce mécanisme ; sans nouvelle sauvegarde, les anciennes restent. Ajouter une limite d'âge et un mécanisme empêchant la réintroduction de données effacées lors d'une restauration. |
| Chiffrement au repos | [src/database/database.js](https://github.com/ZertyAlpha/RPG/blob/d6928e14da779af67a1d66b1766e68dded7e050b/src/database/database.js) : better-sqlite3 standard, WAL, sans clé applicative visible ; backup.js produit des fichiers SQLite | Aucun chiffrement applicatif observé dans ces chemins. Un disque chiffré peut satisfaire ce besoin : son existence et la couverture des backups/logs restent à vérifier sur l'hébergement. Ne pas conclure que les disques sont forcément en clair. |
| Logs | [src/utils/logger.js](https://github.com/ZertyAlpha/RPG/blob/d6928e14da779af67a1d66b1766e68dded7e050b/src/utils/logger.js) : masquage, console, fichiers JSONL et envoi Discord facultatif ; [src/events/ready.js](https://github.com/ZertyAlpha/RPG/blob/d6928e14da779af67a1d66b1766e68dded7e050b/src/events/ready.js) : branchement de LOG_DISCORD_CHANNEL_ID | La purge locale ne couvre pas le salon Discord ni la console/hébergeur. Prévoir leurs durées, accès et suppressions. Les noms/IDs d'acteur et de cible peuvent figurer dans les logs. |
| Délai des logs | logger.js : listExpiredLogFiles compare strictement la date à J-retentionDays ; purge lors de la première écriture d'un nouveau jour UTC ; config.js : LOG_RETENTION_DAYS configurable, 14 par défaut | « Suppression automatique après exactement 14 jours » était trop affirmatif. Le jour limite est encore conservé et une absence d'écritures peut différer la purge. La politique est corrigée. |
| Notifications privées | [src/services/NotificationService.js](https://github.com/ZertyAlpha/RPG/blob/d6928e14da779af67a1d66b1766e68dded7e050b/src/services/NotificationService.js) : user.send sans contrôle d'opt-in dans cette fonction ; [src/services/SessionNotificationService.js](https://github.com/ZertyAlpha/RPG/blob/d6928e14da779af67a1d66b1766e68dded7e050b/src/services/SessionNotificationService.js) : DM en secours après échec du salon | Vérifier le consentement explicite dans les parcours appelants et proposer une préférence/retrait avant les DM. La présente revue ne prouve pas l'absence de toute autorisation ailleurs. Ne pas assimiler une permission OAuth identify à l'accord aux notifications. |
| Accès au légal | Recherches GitHub privacy, confidentialité, kingdom-legal et rgpd sans résultat pertinent | Aucun lien intégré identifié par ces recherches. Vérifier le parcours complet puis ajouter des accès visibles dans /help et l'Activité si absents ; contrôler séparément les URL du Developer Portal. |

## Éléments confirmés dans les chemins inspectés

- [src/index.js](https://github.com/ZertyAlpha/RPG/blob/d6928e14da779af67a1d66b1766e68dded7e050b/src/index.js) : seuls Guilds et GuildMembers sont déclarés, sans MessageContent. Aucun gestionnaire messageCreate trouvé par la recherche ciblée. Cela soutient l'absence de lecture des messages ordinaires, sans remplacer un inventaire exhaustif.
- [activity/client/src/discord.ts](https://github.com/ZertyAlpha/RPG/blob/d6928e14da779af67a1d66b1766e68dded7e050b/activity/client/src/discord.ts) : autorisation OAuth avec scope identify, jeton passé au SDK sans stockage local dans ce fichier.
- [activity/server/auth.js](https://github.com/ZertyAlpha/RPG/blob/d6928e14da779af67a1d66b1766e68dded7e050b/activity/server/auth.js) : échange OAuth, lecture de /users/@me, contrôle du serveur de l'instance et du cache membre, ticket aléatoire 32 octets à usage unique valable 60 secondes ; aucun stockage en base dans cette fonction. La locale est aussi renvoyée au client.
- [activity/server/ActivityServer.js](https://github.com/ZertyAlpha/RPG/blob/d6928e14da779af67a1d66b1766e68dded7e050b/activity/server/ActivityServer.js) : contrôle d'origine WebSocket Discord, limites de taille/débit et de connexions ; événements d'authentification associés à userId, sans corps OAuth journalisé. Ces contrôles ne prouvent pas qu'une requête vient physiquement de Discord : l'authentification et la configuration réseau restent déterminantes.
- [src/events/ready.js](https://github.com/ZertyAlpha/RPG/blob/d6928e14da779af67a1d66b1766e68dded7e050b/src/events/ready.js) : chargement en mémoire des membres du serveur pour les rôles et accès. Vérifier la minimisation pour les membres ne jouant pas.
- [src/database/schema.sql](https://github.com/ZertyAlpha/RPG/blob/d6928e14da779af67a1d66b1766e68dded7e050b/src/database/schema.sql) : données de jeu nombreuses, position/avatar de personnage, musée et nom du donateur, rangs staff et copies restaurables. L'inventaire de la politique doit couvrir ces catégories.
- Les recherches localStorage identifient langue, commandes, audio et mini-carte dans le client. Le texte est complété.
- [src/services/KingdomApiService.js](https://github.com/ZertyAlpha/RPG/blob/d6928e14da779af67a1d66b1766e68dded7e050b/src/services/KingdomApiService.js) : API optionnelle de fortune des Maisons protégée par jeton. Déterminer le destinataire effectif si elle est activée.

## Corrections documentaires réalisées

FR/EN : priorité des règles Discord, signalements, autorisations, usages des données, partage et circonstances de suppression ; résumé des données corrigé ; liens cliquables ; polices Google externes retirées. Après lecture du RPG : précision sur les logs Discord et la console, la purge locale configurable, les sauvegardes par nombre, la suppression de personnage restaurable, l'accès de la modération, la locale OAuth et les préférences locales. Les mécanismes constatés sont décrits sans les présenter comme suffisants.

Les engagements d'effacement ajoutés restent des exigences à mettre en œuvre et à valider avant fusion. Les affirmations techniques préexistantes non auditées ne deviennent pas vraies du seul fait de cette PR.

## Éléments externes ou organisationnels encore à confirmer

- [ ] Hébergeur, prestataires, lieux de traitement et garanties de transferts.
- [ ] Chiffrement des volumes, copies et sauvegardes ; accès effectifs et configuration TLS/proxy. Le serveur applicatif utilise node:http : vérifier la terminaison HTTPS et l'absence d'exposition directe.
- [ ] Identité et coordonnées adéquates de l'éditeur, fondement RGPD et choix préexistant du droit français ; droits impératifs des utilisateurs et autorité compétente.
- [ ] Contact legosi_ accessible après départ du serveur ; relais des modérateurs opérationnel ; suivi des demandes et signalements.
- [ ] Procédure de réponse aux incidents, notification à Discord et aux personnes selon les règles applicables.
- [ ] URL publiques du Developer Portal et accès depuis bot/Activité.
- [ ] Gratuité en production, licences de ressources et classification d'âge requise.
- [ ] Avant toute monétisation : nouvelle revue des obligations Premium Apps.

## Validation

Lecture des fichiers cités et recherches ciblées GitHub, puis comparaison avec la politique. Aucun audit exhaustif des dépendances ou secrets et aucun test d'exécution du bot. Vérification de l'identité des contenus relus depuis GitHub et de l'équilibrage des balises HTML sur les pages modifiées. La PR reste en brouillon et main n'est pas modifiée.
