
### Context Guidance

**Projet TrooPay - Contexte:**
- Web app existante générant des bulletins de paie via flux bancaires
- Système de sécurité actuel: cachet électronique + QR code
- Base de données de bulletins authentiques

**Application Mobile OCR - Vision:**
- Objectif principal: Marketing & effet "wow"
- Fonctionnalité clé: Détection de falsification par comparaison OCR
- Prototype Bubble existant 

**Focus Technique:**
- Reconnaissance de texte (OCR) sur bulletins de paie
- Comparaison avec bulletin en base de données
- Détection d'anomalies (lettres modifiées, chiffres ajoutés)
- Architecture mobile et intégration avec backend TrooPay

---

## Technique Execution Results

### Phase 1: Morphological Analysis - Cartographie Systématique

**Objectif :** Cartographier exhaustivement toutes les dimensions techniques du système OCR TrooPay et identifier les combinaisons optimales.

#### Dimensions Techniques Identifiées et Analysées :

**1. Architecture Mobile**
- **Choix confirmé :** React Native
- **Rationale :** Code partagé iOS/Android, écosystème riche, intégration facile avec backend web TrooPay existant

**2. Moteur OCR**
- **Options explorées :** Google ML Kit on-device, Vision Framework (Apple), Tesseract, Cloud OCR APIs (Google Vision, AWS Textract, Azure)
- **Recommandation :** ML Kit on-device
- **Critères décisifs :** Rapidité (effet wow), traitement local instantané, gratuit, précision suffisante
- **Trade-off accepté :** Légère baisse de précision vs cloud, mais gain majeur en UX

**3. Stratégie de Capture**
- **Options explorées :** Scan QR puis OCR séparés / OCR puis QR / Simultané
- **Choix :** Capture simultanée (QR + texte bulletin en une fois)
- **Rationale :** Fluidité UX maximale, effet wow, simplicité utilisateur

**4. Méthode de Parsing**
- **Insight clé découvert :** Template TrooPay UNIQUE = avantage colossal
- **Options évaluées :** Pixel-perfect / Pattern regex / Zone-based flexible / ML extraction
- **Choix :** **Zone-based flexible (Hybride)**
- **Architecture :** Définir zones de recherche approximatives (en-tête, rémunération, cotisations, net) où chercher les valeurs, avec tolérance aux variations mineures de position
- **Rationale :** Le salaire brut reste dans la zone "rémunération" même s'il bouge de quelques pixels. Balance parfaite entre robustesse et flexibilité.

**5. Stratégie de Comparaison**
- **Clarification majeure :** Comparaison SÉMANTIQUE des valeurs, pas positionnelle au pixel près
- **Principe :** "Peu importe la position, du moment que le salaire indiqué soit le même"
- **Choix :** Extraction de valeurs par zone → Comparaison valeur par valeur
- **Exemple :** OCR trouve "Salaire brut: 3450€" (n'importe où dans la zone) vs DB `{salaire_brut: 3450}` → Comparaison de la valeur numérique uniquement

**6. Format Backend / API**
- **Découverte :** Backend TrooPay déjà fonctionnel avec endpoint GET bulletin
- **Format retour :** JSON structuré
- **Impact :** Comparaison devient triviale - matching direct des valeurs extraites vs JSON
- **Action requise :** Exposer endpoint public/protégé pour app mobile

**7. Quality Control & Validation**
- **Exigence critique :** Détection de flou/mauvais éclairage OBLIGATOIRE
- **Approche :** Validation pré-OCR en temps réel
- **Feedback utilisateur :** "Photo floue, veuillez réessayer" avec guidage
- **Rationale :** Éviter faux positifs dus à mauvaise qualité OCR

**8. Gestion Connectivité**
- **Décision :** Pas de mode offline
- **Rationale :** Connexion réseau = prérequis absolu pour interroger serveurs TrooPay
- **UX :** Message clair "Connexion requise pour vérification"
- **Simplification :** Élimine complexité cache/queue/sync

**9. Logique Métier QR Code**
- **Insight simplificateur :** QR code valide = bulletin EXISTE TOUJOURS en DB (garantie système)
- **Contenu QR :** Lien vers page certification (contient ID bulletin)
- **Impact :** Pas de gestion du cas "QR valide mais bulletin introuvable" - impossible par design
- **Simplification majeure :** Logique d'erreur réduite

#### Matrice des Combinaisons Techniques Optimales :

| Composant | Technologie Retenue | Justification |
|-----------|---------------------|---------------|
| Framework Mobile | React Native | Rapidité dev, code partagé, écosystème |
| Moteur OCR | ML Kit on-device | Instantanéité, effet wow, gratuit |
| Scan QR | Simultané avec OCR | Fluidité UX, une seule capture |
| Parsing | Zone-based flexible | Template unique exploité, tolérance variations |
| Comparaison | Sémantique par valeur | Robustesse, précision non-négociable sur chiffres |
| Backend API | Existant TrooPay + endpoint mobile | Time-to-market réduit, JSON structuré |
| Quality Control | Détection flou pré-OCR | Prévention faux positifs, guidage utilisateur |
| Mode Offline | Bloqué avec message | Simplicité architecture, connexion requise |


#### Architecture Technique Émergente :

```
Flow Utilisateur (focus scan camera):
1. Ouvrir app → deux boutons: Camera / importer 
2. Camera avec overlay guidage
3. Capturer bulletin (validation qualité temps réel)
4. Traitement simultané :
   - Scan QR → Extract bulletin_id
   - OCR zones prédéfinies → Extract valeurs clés
5. API call → GET /api/bulletins/{id} → JSON structuré
6. Comparaison valeur par valeur :
   - ocr_salaire_brut === db_salaire_brut
   - ocr_salaire_net === db_salaire_net
   - ...
6. Résultat :
   - ✅ Toutes valeurs matchent → "Bulletin authentique"
   - ❌ Une valeur diffère → "🚨 FALSIFICATION DÉTECTÉE sur [champ]"
   - ⚠️ Photo floue → "Veuillez reprendre la photo"
   - 📡 Pas de réseau → "Connexion requise"
```

**Énergie et Engagement :** Exploration collaborative hautement productive avec clarifications décisives sur architecture et contraintes métier.

---

### Phase 2: First Principles Thinking - Reconstruction Fondamentale

#### Déconstruction & Révélations Fondamentales :

**🔥 Question Fondamentale #1 : Qu'est-ce qu'une Falsification, Vraiment ?**

**Exploration :** Comment prouver scientifiquement/mathématiquement qu'un bulletin est falsifié ?

**Approches possibles :**
- Comparer le contenu (chiffres, texte)
- Vérifier la source (qui l'a émis)
- Chercher incohérences internes (calculs incorrects)
- Valider signature cryptographique

**Vérité Fondamentale Identifiée :**
> **"La preuve ultime, c'est la source. L'expert comptable appose sa signature sur l'audit et son cachet sur chaque bulletin."**

**Insight critique :** Le cachet électronique de l'expert comptable = preuve d'authenticité cryptographique

---

**💡 Challenge First Principles : Pourquoi l'OCR Alors ?**

**Question provocatrice :** Si le cachet électronique est la preuve ultime, pourquoi faire de l'OCR ?

**Réponse First Principles :**

**Monde Numérique (PDF) :**
- Métadonnées accessibles
- Signature cryptographique vérifiable
- OCR pas strictement nécessaire

**Monde Physique (Papier imprimé) :**
- Pas de métadonnées accessibles
- Pas de signature crypto lisible par machine
- Possibilité de falsification : Photoshop → Réimprimer
- **SEULE SOLUTION : OCR pour reconstruire le numérique depuis le physique**

**Vérité Fondamentale Validée :**
> **L'OCR est le PONT NÉCESSAIRE entre monde physique et monde numérique**

**Cas d'usage réel :**
- Utilisateur présente bulletin papier pour prêt/location/justificatif
- Destinataire veut vérifier authenticité
- App TrooPay OCR : Scanner papier → Vérification instantanée
- **EFFET WOW = Transformer un papier muet en preuve vérifiable !**

---

**🎯 Vérité Fondamentale #1 : La Source de Vérité Absolue**

**Énoncé :** La base de données TrooPay avec signature expert comptable = référence canonique absolue.

**Principe :**
```
DB TrooPay = Source de Vérité
Tout ce qui diffère de la DB = Falsification
```

**Validation :** Si données_OCR === données_DB → Authentique
Si données_OCR ≠ données_DB → Falsifié

**Conséquence architecturale :** Pas besoin de validation de cohérence interne (calculs, formules). La comparaison stricte avec DB suffit.

**Stratégie retenue : Comparaison Stricte DB uniquement**

---

**🔍 Question Fondamentale #3 : Faut-il Tout Vérifier ?**

**Exploration :** Scanner exhaustivement 30-40 lignes du bulletin ou seulement les champs critiques ?

**Hypothèses challengées :**
- OCR exhaustif = plus sûr mais plus lent
- OCR ciblé = plus rapide mais risqué ?

**Vérité Fondamentale Identifiée :**
> **Tous les champs de DONNÉES doivent être vérifiés. Pas les labels statiques.**

**Champs Critiques à Vérifier :**
- ✅ Identité chef d'entreprise
- ✅ Nom entreprise
- ✅ SIRET
- ✅ Tous les montants financiers (revenus, charges, net, etc.)
- ✅ Période
- ❌ PAS les labels statiques ("Bulletin certifié:", "Salaire net:", titres, etc.)

**Rationale :** Vérification complète nécessaire pour certification robuste. Les falsifications peuvent toucher n'importe quel champ de données.

---

**🧠 Question Fondamentale #4 : Qu'est-ce qu'un "Match" ?**

**Problématique :** Définir l'égalité au niveau fondamental.

**Scénario :**
- DB contient : `"6500"`
- OCR lit : `"6 500€"` (espace + symbole)
- Est-ce un match ou une falsification ?

**Options explorées :**

**A) Match Strict (caractère par caractère)**
- `"6500" === "6 500"` → FALSE → Faux positif

**B) Match Normalisé (nettoyage puis comparaison)**
- `normalize("6 500€")` → `"6500"`
- `"6500" === "6500"` → TRUE
- Tolérant aux variations OCR normales

**C) Match Sémantique (extraction valeur)**
- `extract_number("6 500€")` → `6500`
- Compare valeurs numériques

**Vérité Fondamentale Retenue :**
> **Normalisation intelligente nécessaire pour distinguer vraie falsification vs variation technique**

**Stratégie de Normalisation :**
- **Nombres :** Enlever espaces, symboles → Comparer valeurs numériques pures
- **Texte :** Normalisation minimale (casse uniforme, espaces multiples) → Comparer chaînes

**Principe :** Tolérer variations OCR normales SANS compromettre détection de vraies modifications.

---

**📄 Révélation #5 : Mode Import PDF**

**Découverte :** Deux modes d'entrée nécessaires :

**Mode 1 : Photo Papier (Principal)**
```
Photo → OCR visuel → Extraction données → Comparaison DB
```

**Mode 2 : Import PDF**
```
PDF → Extraction texte/OCR → Extraction données → Comparaison DB
```

**Question First Principles :** PDF = texte extractible direct ou OCR visuel ?

**Réponse :** Dépend du cas d'usage :
- PDF original TrooPay : Extraction texte direct possible
- PDF scanné (papier → PDF) : OCR visuel nécessaire

**Architecture retenue :** Support des deux modes avec logique adaptative.

---

#### Vérités Fondamentales Validées (Synthèse) :


**✅ Vérité #1 : Problème Réel**
- Pont physique↔numérique nécessaire
- Bulletin papier = muet, OCR = seule solution viable

**✅ Vérité #2 : Source de Vérité**
- DB TrooPay = référence absolue
- Comparaison stricte suffit (pas besoin validation cohérence interne)

**✅ Vérité #3 : Vérification Complète**
- Tous champs de données critiques (identité, entreprise, montants, période)
- Pas les labels statiques

**✅ Vérité #4 : Normalisation Intelligente**
- Nombres : normalisation complète
- Texte : normalisation minimale
- Balance tolérance OCR vs sécurité

**✅ Vérité #5 : Deux Modes d'Entrée**
- Photo papier (principal, OCR visuel)
- Import PDF (bonus, extraction texte ou OCR adaptatif)

#### Reconstruction Architecturale Depuis First Principles :

**Objectif fondamental :** Vérifier authenticité bulletin papier TrooPay

**Composants nécessaires minimum :**
1. Identifiant bulletin → QR code (déjà présent) ✅
2. Extraction données papier → OCR visuel ✅
3. Communication DB → API REST (backend existant) ✅
4. Comparaison intelligente → Normalisation + equality check ✅

**Architecture minimale viable validée. Tout le reste = optimisation et UX.**

**Énergie et Engagement :** Déconstruction intense et productive. Révélations majeures sur l'innovation TrooPay et clarifications fondamentales sur normalisation et stratégie de comparaison.

---

### Phase 3: Solution Matrix - Plan d'Action Concret

**Objectif :** Créer une grille décisionnelle systématique croisant composants techniques avec critères d'évaluation pour converger vers architecture finale et plan d'implémentation.

#### Solution Matrix - Décisions Techniques Finales

**Critères d'Évaluation :** Rapidité (effet wow), Précision (non-négociable), Coût, Maintenabilité, Time-to-Market

**Composant 1 : Framework Mobile**
- **Décision : React Native (Bare)** - Score 22/25
- Justification : Code partagé iOS/Android, écosystème riche OCR/Camera, intégration backend TrooPay, intuition initiale validée
- Alternative rejetée : Native (17/25 - double dev), Flutter (20/25 - moins d'expérience équipe)

**Composant 2 : Moteur OCR**
- **Décision : ML Kit on-device** - Score 23/25
- Justification : Traitement instantané local (pas latence réseau), gratuit, précision suffisante bulletins structurés
- Package : `@react-native-ml-kit/text-recognition` ou `react-native-mlkit-text-recognition`
- Trade-off accepté : Précision légèrement < cloud, mais gain UX >> perte précision

**Composant 3 : Stratégie Scan QR + OCR**
- **Décision : Simultané (1 capture)** - Score 21/25
- Justification : UX optimale, une seule photo, moins de friction, effet wow maximal
- Package : `react-native-vision-camera` + détection QR intégrée + ML Kit
- Workflow : Capture → Traiter QR || OCR en parallèle → Résultat combiné

**Composant 4 : Architecture Parsing & Extraction**
- **Décision : Zone-based flexible** - Score 20/25
- Justification : Template TrooPay unique exploité, zones approximatives (en-tête, rémunération, cotisations, net), tolérant micro-variations
- Implémentation : Définir zones par % hauteur document, pattern matching dans zones
```javascript
const ZONES = {
  header: { y: 0, height: 0.15 },
  remuneration: { y: 0.15, height: 0.30 },
  cotisations: { y: 0.45, height: 0.30 },
  net: { y: 0.75, height: 0.25 }
}
```

**Composant 5 : Stratégie de Comparaison**
- **Décision : Normalisée intelligente** - Score 23/25
- Justification : Balance tolérance variations OCR / détection vraies falsifications
- Implémentation :
  - Nombres : Enlever espaces, symboles → Comparer valeurs numériques pures
  - Texte : Normalisation minimale (casse, espaces multiples) → Comparer chaînes

**Composant 6 : Backend API**
- **Décision : Endpoint mobile dédié TrooPay** - Score 23/25
- Justification : Backend existant étendu, time-to-market optimal, JSON structuré optimisé mobile
- Endpoint : `GET /api/mobile/bulletins/verify?qr_id=xxx`
- Retour : JSON avec champs critiques uniquement (identité, entreprise, financier, période)

**Composant 7 : Quality Control**
- **Décision : Hybride Pré-OCR + Post-OCR** - Score 20/25
- Pré-OCR : Détection sharpness/blur temps réel (guidage utilisateur)
- Post-OCR : Vérification confidence scores ML Kit (< 80% → retake)

**Composant 8 : Mode Import PDF**
- **Décision : Feature V2 - Extraction texte + OCR fallback** - Score 15/20
- Logique adaptative : Essai extraction texte direct → Si échec render + OCR visuel
- Différé post-MVP

#### Stack Technique Final Retenu

| Composant | Technologie | Package/Outil | Justification |
|-----------|-------------|---------------|---------------|
| Framework Mobile | React Native Bare | react-native@latest | Code partagé, écosystème |
| Moteur OCR | ML Kit on-device | @react-native-ml-kit/text-recognition | Instantané, gratuit |
| Camera/Scan | Vision Camera | react-native-vision-camera | Scan simultané QR+OCR |
| QR Decoder | Vision Camera QR | Intégré vision-camera | Performance optimale |
| Parsing | Zone-based + Regex | Code custom | Template unique exploité |
| Comparaison | Normalisation intelligente | Code custom | Balance tolérance/sécurité |
| Backend API | Endpoint mobile TrooPay | Extension backend existant | Time-to-market |
| Quality Control | Pré+Post OCR | Laplacian variance + ML confidence | Guidage + fiabilité |
| State Management | Context API ou Zustand | react-context / zustand | Simplicité suffisante |
| Navigation | React Navigation | @react-navigation/native | Standard industrie |
| HTTP Client | Axios | axios | Simple et robuste |

---

## 🚀 PLAN D'IMPLÉMENTATION DÉTAILLÉ

### Stratégie de Livraison : MVP → V1 → V2

**Philosophie :** Itérations rapides avec validation utilisateur à chaque phase.

---

## 📦 PHASE MVP (Minimum Viable Product) - 3-4 semaines

**Objectif :** Valider le concept avec fonctionnalité core = Scan bulletin papier + Vérification authenticité

### Sprint 1 : Foundation & Setup (5 jours)

**User Story 1.1 :** En tant que développeur, je veux setup le projet React Native pour commencer le développement
- **Tâches :**
  - [ ] Init projet React Native (Bare) : `npx react-native init TroopayOCR`
  - [ ] Config ESLint + Prettier
  - [ ] Setup structure dossiers (screens, components, services, utils, constants)
  - [ ] Config Git + .gitignore
  - [ ] Install dépendances core : `react-navigation`, `axios`, `zustand`
  - [ ] Config environnements (dev, staging, prod) avec `react-native-config`
  - [ ] Setup Fastlane (iOS/Android build automation)
- **Estimation :** 2 jours
- **Risques :** Problèmes config natifs (Xcode, Android Studio)
- **Mitigation :** Doc officielle React Native + troubleshooting

**User Story 1.2 :** En tant que développeur, je veux intégrer la caméra et ML Kit pour préparer le scan
- **Tâches :**
  - [ ] Install `react-native-vision-camera` : `npm install react-native-vision-camera`
  - [ ] Config permissions iOS (Info.plist) : `NSCameraUsageDescription`
  - [ ] Config permissions Android (AndroidManifest.xml) : `CAMERA`
  - [ ] Install `@react-native-ml-kit/text-recognition`
  - [ ] Config ML Kit Android (build.gradle)
  - [ ] Créer composant `<CameraScreen />` basique avec preview
  - [ ] Test capture photo sur device réel
- **Estimation :** 2 jours
- **Risques :** Permissions refusées, ML Kit config complexe
- **Mitigation :** Tester sur devices réels iOS + Android dès J1

**User Story 1.3 :** En tant que backend dev, je veux créer l'endpoint mobile API
- **Tâches :**
  - [ ] Analyser endpoint GET bulletin existant TrooPay
  - [ ] Créer `GET /api/mobile/bulletins/verify?qr_id={id}`
  - [ ] Response JSON structuré (chef_entreprise, entreprise, financier, periode)
  - [ ] Gérer erreurs (bulletin non trouvé, QR invalide)
  - [ ] Auth : API Key ou JWT (selon archi TrooPay)
  - [ ] Rate limiting (eviter abuse)
  - [ ] Déployer sur environnement staging
  - [ ] Documenter API (Swagger/Postman)
- **Estimation :** 1 jour
- **Risques :** Dépendances backend TrooPay existant
- **Mitigation :** Coordination avec équipe backend TrooPay

---

### Sprint 2 : Core Feature - Scan & OCR (5 jours)

**User Story 2.1 :** En tant qu'utilisateur, je veux scanner un bulletin avec ma caméra
- **Tâches :**
  - [ ] Créer UI `<ScanScreen />` avec overlay guidage
  - [ ] Ajouter bouton capture photo
  - [ ] Afficher preview photo capturée
  - [ ] Gérer états : idle, capturing, captured, processing
  - [ ] Design overlay : cadre bulletin + instructions
  - [ ] Feedback visuel capture (flash, animation)
- **Estimation :** 2 jours
- **Critères Acceptance :**
  - [ ] Caméra s'ouvre sans erreur
  - [ ] Photo capturée est nette et visible
  - [ ] UI intuitive et guidante

**User Story 2.2 :** En tant que système, je veux extraire texte du bulletin via OCR
- **Tâches :**
  - [ ] Créer service `OCRService.js`
  - [ ] Fonction `recognizeText(imageUri)` avec ML Kit
  - [ ] Parser résultats OCR (blocks, lignes, confidence)
  - [ ] Filtrer résultats par confidence > 70%
  - [ ] Logger résultats OCR (debug)
  - [ ] Gérer erreurs OCR (timeout, échec)
- **Estimation :** 1.5 jours
- **Critères Acceptance :**
  - [ ] OCR retourne texte structuré
  - [ ] Temps traitement < 3 secondes
  - [ ] Confidence scores disponibles

**User Story 2.3 :** En tant que système, je veux détecter et scanner le QR code du bulletin
- **Tâches :**
  - [ ] Activer QR code scanning dans `vision-camera`
  - [ ] Config frame processor pour QR detection
  - [ ] Extraire URL/ID du QR code
  - [ ] Parser URL pour récupérer `bulletin_id`
  - [ ] Gérer cas : QR non détecté, QR invalide
  - [ ] Simultanéité : QR scan || OCR (parallèle)
- **Estimation :** 1.5 jours
- **Critères Acceptance :**
  - [ ] QR code détecté en < 1 seconde
  - [ ] ID bulletin extrait correctement
  - [ ] Gestion erreurs robuste

---

### Sprint 3 : Parsing & Comparison Logic (5 jours)

**User Story 3.1 :** En tant que système, je veux extraire les champs critiques depuis le texte OCR
- **Tâches :**
  - [ ] Créer service `ParsingService.js`
  - [ ] Définir constantes ZONES (header, remuneration, cotisations, net)
  - [ ] Fonction `extractInZone(ocrBlocks, zone, pattern)`
  - [ ] Patterns regex pour champs critiques :
    - Nom : `/(Nom|Name).*?([A-Z][a-z]+)/i`
    - SIRET : `/SIRET.*?(\d{14})/`
    - Revenu brut : `/Revenu brut.*?([\d\s€]+)/i`
    - Revenu net : `/Revenu net.*?([\d\s€]+)/i`
    - Période : `/(Janvier|Février|...|Décembre)\s*(\d{4})/i`
  - [ ] Tester parsing sur 10 bulletins TrooPay réels
  - [ ] Ajuster patterns selon résultats
- **Estimation :** 3 jours
- **Critères Acceptance :**
  - [ ] 90%+ champs extraits correctement sur échantillon test
  - [ ] Gestion cas : champ non trouvé

**User Story 3.2 :** En tant que système, je veux comparer les données OCR avec la DB
- **Tâches :**
  - [ ] Créer service `ComparisonService.js`
  - [ ] Fonction `normalizeNumber(str)` : enlever espaces, €, etc.
  - [ ] Fonction `normalizeText(str)` : lowercase, trim, espaces multiples
  - [ ] Fonction `compareFields(ocrData, dbData)`
  - [ ] Retourner résultat : `{ match: true/false, differences: [] }`
  - [ ] Logger toutes comparaisons (analytics)
  - [ ] Tests unitaires (Jest) pour normalisation
- **Estimation :** 2 jours
- **Critères Acceptance :**
  - [ ] Comparaison tolère variations OCR normales
  - [ ] Détecte falsifications (changement chiffre)
  - [ ] Tests unitaires passent 100%

---

### Sprint 4 : Integration & UX Polish (5 jours)

**User Story 4.1 :** En tant qu'utilisateur, je veux voir le résultat de la vérification
- **Tâches :**
  - [ ] Créer `<ResultScreen />`
  - [ ] UI Authentique : ✅ icône + "Bulletin Authentique"
  - [ ] UI Falsifié : 🚨 icône + "Falsification Détectée" + champs différents
  - [ ] UI Erreur : ⚠️ messages (réseau, OCR échec, QR invalide)
  - [ ] Bouton "Télécharger bulletin authentique" (si match)
  - [ ] Bouton "Accéder à la certification" (lien page TrooPay)
  - [ ] Bouton "Scanner un autre bulletin"
  - [ ] Animations transitions (lottie)
- **Estimation :** 2 jours
- **Critères Acceptance :**
  - [ ] Résultat clair et compréhensible
  - [ ] Actions disponibles fonctionnelles

**User Story 4.2 :** En tant qu'utilisateur, je veux avoir un guidage si ma photo est floue
- **Tâches :**
  - [ ] Créer `QualityControlService.js`
  - [ ] Fonction `isImageSharp(imageUri)` : Laplacian variance
  - [ ] Si flou détecté : overlay "Photo floue, rapprochez-vous"
  - [ ] Post-OCR : vérifier avg confidence ML Kit
  - [ ] Si < 80% : "Qualité insuffisante, veuillez réessayer"
  - [ ] Feedback temps réel pendant capture
- **Estimation :** 1.5 jours
- **Critères Acceptance :**
  - [ ] Détection flou > 80% précision
  - [ ] Messages guidage clairs
  - [ ] Réduction faux positifs

**User Story 4.3 :** En tant que développeur, je veux tester le flow complet end-to-end
- **Tâches :**
  - [ ] Tester sur 20 bulletins TrooPay réels (papier)
  - [ ] Mesurer temps scan → résultat (< 5 sec objectif)
  - [ ] Tester falsifications intentionnelles (chiffres modifiés)
  - [ ] Tester edge cases (mauvais éclairage, bulletin plié, etc.)
  - [ ] Fixer bugs critiques
  - [ ] Préparer démo stakeholders
- **Estimation :** 1.5 jours
- **Critères Acceptance :**
  - [ ] Flow complet fonctionne sans crash
  - [ ] Détection falsification 100% sur tests contrôlés
  - [ ] Performance acceptable

---

### Livrables MVP (Fin Sprint 4)

✅ **Fonctionnalités :**
- Scan bulletin papier via caméra
- OCR automatique + scan QR simultané
- Extraction champs critiques
- Comparaison avec DB TrooPay
- Résultat : Authentique ✅ ou Falsifié 🚨
- Guidage qualité photo
- Accès page certification

✅ **Screens :**
1. Home (bouton "Scanner bulletin")
2. Camera/Scan (avec overlay guidage)
3. Processing (loader avec feedback)
4. Result (authentique/falsifié + actions)

✅ **Backend :**
- Endpoint mobile API fonctionnel
- JSON structuré optimisé

✅ **Tests :**
- Tests unitaires services (parsing, comparaison)
- Tests end-to-end sur bulletins réels

**Validation MVP :** Démo interne + Tests utilisateurs (5-10 entrepreneurs)

---

## 📦 PHASE V1 (Version 1) - 2-3 semaines

**Objectif :** Enrichir l'expérience avec historique, améliorations UX, analytics

### Sprint 5 : Historique & Persistence (5 jours)

**User Story 5.1 :** En tant qu'utilisateur, je veux voir l'historique de mes vérifications
- **Tâches :**
  - [ ] Setup AsyncStorage ou MMKV (storage local)
  - [ ] Créer `StorageService.js`
  - [ ] Sauvegarder chaque vérification : date, résultat, bulletin_id, thumbnail
  - [ ] Créer `<HistoryScreen />` avec liste
  - [ ] Design carte historique : date + résultat + aperçu
  - [ ] Tap carte → détails complets
  - [ ] Bouton supprimer historique (confirmation)
  - [ ] Navigation vers History depuis Home
- **Estimation :** 3 jours
- **Critères Acceptance :**
  - [ ] Historique persiste entre sessions
  - [ ] Liste triée par date DESC
  - [ ] Performance fluide (100+ entrées)

**User Story 5.2 :** En tant qu'utilisateur, je veux partager le résultat de vérification
- **Tâches :**
  - [ ] Bouton "Partager" sur ResultScreen
  - [ ] Générer screenshot ou PDF résultat
  - [ ] Share API native (`react-native-share`)
  - [ ] Formats : Image, PDF, Texte
  - [ ] Ajouter watermark TrooPay (optionnel)
- **Estimation :** 2 jours
- **Critères Acceptance :**
  - [ ] Partage fonctionnel iOS + Android
  - [ ] Formats supportés

---

### Sprint 6 : UX Enhancements & Onboarding (4 jours)

**User Story 6.1 :** En tant que nouvel utilisateur, je veux comprendre comment utiliser l'app
- **Tâches :**
  - [ ] Créer `<OnboardingScreen />` (première ouverture)
  - [ ] 3-4 slides explicatifs avec illustrations
  - [ ] Slide 1 : "Vérifiez vos bulletins TrooPay"
  - [ ] Slide 2 : "Scannez en un instant"
  - [ ] Slide 3 : "Détection automatique de falsification"
  - [ ] Slide 4 : Permissions (caméra)
  - [ ] Skip onboarding (bouton)
  - [ ] Mémoriser onboarding vu (AsyncStorage)
- **Estimation :** 2 jours
- **Critères Acceptance :**
  - [ ] Onboarding clair et engageant
  - [ ] Affiché seulement première fois

**User Story 6.2 :** En tant qu'utilisateur, je veux une expérience fluide et polie
- **Tâches :**
  - [ ] Animations transitions écrans (react-native-reanimated)
  - [ ] Loading states avec skeletons
  - [ ] Feedback haptique (vibrations) sur succès/échec
  - [ ] Dark mode support (optionnel)
  - [ ] Améliorer accessibilité (labels, contrast)
  - [ ] Gérer offline gracefully (message clair)
  - [ ] Error boundaries (éviter crashes)
- **Estimation :** 2 jours
- **Critères Acceptance :**
  - [ ] App fluide 60fps
  - [ ] Pas de crashes sur erreurs réseau
  - [ ] Feedback utilisateur constant

---

### Sprint 7 : Analytics & Monitoring (3 jours)

**User Story 7.1 :** En tant que product owner, je veux tracker l'usage de l'app
- **Tâches :**
  - [ ] Intégrer Firebase Analytics ou Mixpanel
  - [ ] Events : `scan_initiated`, `scan_completed`, `scan_failed`, `verification_authentic`, `verification_falsified`
  - [ ] Properties : temps scan, confidence OCR, type erreur
  - [ ] Screen tracking automatique
  - [ ] User properties (anonyme)
- **Estimation :** 1.5 jours
- **Critères Acceptance :**
  - [ ] Events loggés correctement
  - [ ] Dashboard analytics accessible

**User Story 7.2 :** En tant que développeur, je veux monitorer crashes et erreurs
- **Tâches :**
  - [ ] Intégrer Sentry ou Bugsnag
  - [ ] Capturer crashes automatiquement
  - [ ] Logger erreurs API, OCR, parsing
  - [ ] Source maps upload (symbolication)
  - [ ] Alerts Slack sur erreurs critiques
- **Estimation :** 1.5 jours
- **Critères Acceptance :**
  - [ ] Crashes reportés avec stack trace
  - [ ] Taux crash < 1%

---

### Livrables V1 (Fin Sprint 7)

✅ **Nouvelles fonctionnalités :**
- Historique vérifications avec persistence
- Partage résultats (image, PDF, texte)
- Onboarding première utilisation
- Animations et feedback haptique
- Dark mode (optionnel)
- Analytics usage
- Monitoring crashes

✅ **Screens ajoutés :**
- Onboarding (4 slides)
- History (liste + détails)

✅ **Amélioration UX :**
- Transitions fluides
- Loading states élégants
- Gestion offline graceful
- Accessibilité améliorée

**Validation V1 :** Beta test externe (50-100 utilisateurs entrepreneurs)

---

## 📦 PHASE V2 (Version 2) - 2-3 semaines

**Objectif :** Features avancées (import PDF, batch scan, amélioration précision)

### Sprint 8 : Import PDF (5 jours)

**User Story 8.1 :** En tant qu'utilisateur, je veux importer un bulletin PDF depuis mes fichiers
- **Tâches :**
  - [ ] Créer bouton "Importer bulletin" sur Home
  - [ ] Intégrer `react-native-document-picker`
  - [ ] Filtrer fichiers PDF uniquement
  - [ ] Créer `PDFService.js`
  - [ ] Essayer extraction texte direct (react-native-pdf-lib)
  - [ ] Si échec → Render PDF en image → OCR visuel
  - [ ] Reuse parsing et comparaison logic existante
  - [ ] UI : Progress indicator import
- **Estimation :** 3 jours
- **Critères Acceptance :**
  - [ ] Import PDF fonctionnel
  - [ ] Extraction texte ou OCR adaptatif
  - [ ] Même résultat que scan photo

**User Story 8.2 :** En tant que système, je veux optimiser la précision OCR
- **Tâches :**
  - [ ] Pre-processing image : améliorer contraste, netteté
  - [ ] Rotation auto (détection orientation)
  - [ ] Crop automatique bulletin (détection bordures)
  - [ ] Binarisation adaptative
  - [ ] A/B test avec/sans pre-processing
- **Estimation :** 2 jours
- **Critères Acceptance :**
  - [ ] Précision OCR +10% vs baseline
  - [ ] Temps traitement reste < 5 sec

---

### Sprint 9 : Features Avancées (5 jours)

**User Story 9.1 :** En tant qu'utilisateur, je veux scanner plusieurs bulletins d'affilée
- **Tâches :**
  - [ ] Mode "Scan batch" (toggle)
  - [ ] Après scan → Résultat rapide → Retour auto caméra
  - [ ] Compteur bulletins scannés
  - [ ] Résumé batch à la fin
  - [ ] Export batch results (CSV, Excel)
- **Estimation :** 2 jours
- **Critères Acceptance :**
  - [ ] Scan 10+ bulletins sans friction
  - [ ] Export résumé fonctionnel

**User Story 9.2 :** En tant qu'utilisateur admin, je veux un dashboard analytics
- **Tâches :**
  - [ ] Créer `<AdminDashboard />` (optionnel, web ou in-app)
  - [ ] Métriques : total scans, taux authentique/falsifié, temps moyen
  - [ ] Graphiques (react-native-chart-kit)
  - [ ] Export rapports
- **Estimation :** 3 jours
- **Critères Acceptance :**
  - [ ] Dashboard lisible et utile
  - [ ] Données temps réel

---

### Sprint 10 : Polish & App Store Prep (4 jours)

**User Story 10.1 :** En tant que développeur, je veux préparer l'app pour publication
- **Tâches :**
  - [ ] App icon design (1024x1024)
  - [ ] Splash screen design
  - [ ] Screenshots App Store (tous devices)
  - [ ] Description App Store (FR + EN)
  - [ ] Privacy policy + Terms of service
  - [ ] Config signing (iOS + Android)
  - [ ] Build production (release mode)
  - [ ] TestFlight beta (iOS)
  - [ ] Google Play Internal Testing (Android)
- **Estimation :** 2 jours
- **Critères Acceptance :**
  - [ ] Builds signés et uploadés
  - [ ] Metadata complète

**User Story 10.2 :** En tant que QA, je veux valider la release candidate
- **Tâches :**
  - [ ] Plan test exhaustif (checklist)
  - [ ] Tests manuels iOS (3+ devices)
  - [ ] Tests manuels Android (3+ devices)
  - [ ] Tests régression (toutes features)
  - [ ] Performance testing (temps scan, mémoire, batterie)
  - [ ] Fixer bugs bloquants
  - [ ] Validation stakeholders finale
- **Estimation :** 2 jours
- **Critères Acceptance :**
  - [ ] 0 bugs critiques
  - [ ] Performance benchmarks atteints
  - [ ] Validation OK pour release

---

### Livrables V2 (Fin Sprint 10)

✅ **Nouvelles fonctionnalités :**
- Import PDF avec extraction intelligente
- Pre-processing images (amélioration précision)
- Mode scan batch
- Dashboard analytics (optionnel)
- Export résultats batch

✅ **Production ready :**
- App Store / Play Store assets complets
- Privacy policy + ToS
- Builds signés release
- Tests exhaustifs validés

**Validation V2 :** Release publique App Store + Google Play Store

---

## 📊 ARCHITECTURE TECHNIQUE DÉTAILLÉE

### Structure Projet

```
TroopayOCR/
├── src/
│   ├── screens/
│   │   ├── HomeScreen.js
│   │   ├── ScanScreen.js
│   │   ├── ProcessingScreen.js
│   │   ├── ResultScreen.js
│   │   ├── HistoryScreen.js
│   │   ├── OnboardingScreen.js
│   │   └── AdminDashboard.js (V2)
│   ├── components/
│   │   ├── CameraOverlay.js
│   │   ├── ResultCard.js
│   │   ├── HistoryCard.js
│   │   ├── QualityIndicator.js
│   │   └── LoadingSpinner.js
│   ├── services/
│   │   ├── OCRService.js
│   │   ├── QRService.js
│   │   ├── ParsingService.js
│   │   ├── ComparisonService.js
│   │   ├── QualityControlService.js
│   │   ├── APIService.js
│   │   ├── StorageService.js
│   │   ├── PDFService.js (V2)
│   │   └── AnalyticsService.js
│   ├── utils/
│   │   ├── normalizers.js
│   │   ├── validators.js
│   │   ├── formatters.js
│   │   └── constants.js
│   ├── navigation/
│   │   └── AppNavigator.js
│   ├── store/
│   │   └── useStore.js (Zustand)
│   └── assets/
│       ├── images/
│       ├── fonts/
│       └── animations/
├── android/
├── ios/
├── __tests__/
│   ├── services/
│   ├── components/
│   └── utils/
├── package.json
└── README.md
```

---

## ⚠️ RISQUES & MITIGATIONS

### Risques Techniques

| Risque | Probabilité | Impact | Mitigation |
|--------|-------------|--------|------------|
| **Précision OCR insuffisante** | Moyen | Élevé | Pre-processing images, tests exhaustifs, fallback manuel |
| **Performance ML Kit lente** | Faible | Moyen | Optimisation images, threading, loader UX |
| **QR code non détecté** | Moyen | Moyen | Guidage utilisateur, retry automatique, timeout |
| **Variations template bulletin** | Faible | Élevé | Zone-based flexible, tests sur échantillon large |
| **Backend API instable** | Faible | Élevé | Retry logic, cache, mode dégradé |
| **Permissions caméra refusées** | Moyen | Critique | Onboarding explicatif, deeplink settings |
| **Compatibilité devices** | Moyen | Moyen | Tests sur 10+ devices différents, min SDK version |
| **App Store rejection** | Faible | Élevé | Review guidelines strict, privacy policy clair |

### Risques Business

| Risque | Probabilité | Impact | Mitigation |
|--------|-------------|--------|------------|
| **Adoption faible** | Moyen | Élevé | Marketing pré-launch, onboarding engageant, valeur claire |
| **Coût infrastructure** | Faible | Moyen | OCR on-device = gratuit, monitoring coûts API |
| **Concurrence** | Moyen | Moyen | Focus innovation TrooPay unique, time-to-market rapide |
| **Feedback négatif précision** | Moyen | Élevé | Beta tests larges, amélioration continue, support réactif |

---

## ✅ CHECKLIST DÉPLOIEMENT

### Pre-Launch

- [ ] Tests exhaustifs iOS (iPhone 12, 13, 14, SE)
- [ ] Tests exhaustifs Android (Samsung, Pixel, OnePlus)
- [ ] Tests edge cases (bulletins pliés, éclairage faible, etc.)
- [ ] Performance benchmarks atteints (scan < 5 sec)
- [ ] Taux crash < 0.5%
- [ ] Privacy policy rédigée et publiée
- [ ] Terms of service rédigés
- [ ] App icon finalisé (tous formats)
- [ ] Screenshots App Store (6+ par device)
- [ ] Description store optimisée SEO
- [ ] Video preview (optionnel mais recommandé)
- [ ] Validation légale (RGPD, données personnelles)
- [ ] Accord expert comptable TrooPay (utilisation signature)

### Launch

- [ ] Build production signés (iOS + Android)
- [ ] Upload App Store Connect
- [ ] Upload Google Play Console
- [ ] Soumettre review App Store
- [ ] Release Google Play (phased rollout 10%)
- [ ] Monitoring actif (Sentry, Firebase)
- [ ] Support client prêt (FAQ, chatbot)
- [ ] Communiqué presse (optionnel)
- [ ] Social media campaign

### Post-Launch

- [ ] Monitor analytics J+1, J+7, J+30
- [ ] Collecter feedback utilisateurs
- [ ] Fixer bugs critiques < 48h
- [ ] Itérations rapides (release 1-2 semaines)
- [ ] A/B tests features
- [ ] Expansion marketing

---

## 📈 MÉTRIQUES DE SUCCÈS

### Techniques
- **Temps scan moyen** : < 5 secondes (objectif < 3 sec)
- **Taux succès OCR** : > 95%
- **Précision détection** : 100% falsifications intentionnelles
- **Taux crash** : < 0.5%
- **Performance** : 60 fps constant

### Business
- **Téléchargements J+30** : 1000+ (ajuster selon marché)
- **Taux activation** : > 60% (téléchargement → 1er scan)
- **Taux rétention J+7** : > 40%
- **NPS (Net Promoter Score)** : > 50
- **Avis App Store** : > 4.5/5

### Utilisateur
- **Temps onboarding** : < 2 min jusqu'à 1er scan
- **Taux succès 1er scan** : > 80%
- **Scans par utilisateur actif** : > 3/mois
- **Taux partage résultats** : > 20%

---

## 🎯 PROCHAINES ÉTAPES IMMÉDIATES

**Semaine 1 (Démarrage) :**
1. **Validation stakeholders** sur ce plan (1 jour)
2. **Setup projet React Native** (1 jour)
3. **Config CI/CD** (Fastlane, GitHub Actions) (1 jour)
4. **Coordination backend** : Créer endpoint mobile API (1 jour)
5. **Install dépendances** : Vision Camera + ML Kit (1 jour)

**Semaine 2 (Sprint 1 suite) :**
- Premiers tests caméra + OCR sur device réel
- Prototype scan basique fonctionnel

**Milestone 1 (Fin Sprint 2 - J14) :**
- Scan + OCR + QR fonctionnels
- Démo interne équipe

**Milestone 2 (Fin Sprint 4 - J28) :**
- MVP complet fonctionnel
- Tests utilisateurs internes

**Go/No-Go MVP (J30) :**
- Décision lancement Beta externe ou itérations

---

**Énergie et Engagement :** Plan d'implémentation structuré et actionnable avec roadmap claire MVP → V1 → V2, estimations réalistes, et mitigations des risques identifiés.

---
