<!DOCTYPE html>
<html lang="fr">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1" />
<title>Des Tana’s - Concours créatifs</title>
<style>
  @import url('https://fonts.googleapis.com/css2?family=Russo+One&display=swap');

  /* Couleurs */
  :root {
    --yellow: #FFD300;
    --black: #0D0D0D;
    --gray-dark: #222;
    --gray-light: #555;
  }

  body {
    margin: 0;
    background-color: var(--black);
    color: white;
    font-family: 'Russo One', sans-serif;
    display: flex;
    flex-direction: column;
    align-items: center;
    min-height: 100vh;
  }

  header {
    width: 100%;
    background: black;
    padding: 1.5rem 2rem;
    border-bottom: 3px solid var(--yellow);
    text-align: center;
    font-size: 3rem;
    letter-spacing: 0.15em;
    color: var(--yellow);
    text-shadow: 0 0 5px #FFD300AA;
    user-select: none;
  }

  main {
    flex: 1;
    width: 100%;
    max-width: 700px;
    padding: 2rem;
  }

  .theme {
    font-size: 1.5rem;
    margin-bottom: 2rem;
    text-align: center;
    padding: 1rem;
    border: 2px solid var(--yellow);
    border-radius: 12px;
    box-shadow: 0 0 15px var(--yellow);
  }

  form {
    display: flex;
    flex-direction: column;
    gap: 1rem;
    margin-bottom: 3rem;
  }

  label {
    font-weight: 700;
    color: var(--yellow);
  }

  input[type="file"] {
    color: var(--yellow);
  }

  textarea {
    resize: vertical;
    min-height: 80px;
    padding: 0.5rem;
    border-radius: 8px;
    border: 2px solid var(--yellow);
    background: var(--gray-dark);
    color: white;
    font-family: 'Russo One', sans-serif;
    font-size: 1rem;
  }

  button {
    background-color: var(--yellow);
    color: var(--black);
    border: none;
    padding: 1rem;
    font-weight: 700;
    font-size: 1.2rem;
    border-radius: 10px;
    cursor: pointer;
    transition: background-color 0.3s ease;
  }

  button:hover {
    background-color: #e0be00;
  }

  .participations {
    display: flex;
    flex-direction: column;
    gap: 1.5rem;
  }

  .post {
    background: var(--gray-dark);
    border-radius: 12px;
    padding: 1rem;
    box-shadow: 0 0 10px #ffd300aa;
  }

  .post img {
    max-width: 100%;
    border-radius: 10px;
    margin-bottom: 0.5rem;
    object-fit: contain;
  }

  .post p {
    white-space: pre-wrap;
    font-size: 1rem;
    color: var(--yellow);
  }

  .winner {
    margin-top: 2rem;
    text-align: center;
    font-size: 1.8rem;
    font-weight: 700;
    color: #00ff00;
    text-shadow: 0 0 5px #00ff00aa;
  }

  footer {
    margin-top: auto;
    padding: 1rem;
    color: var(--gray-light);
    font-size: 0.8rem;
    user-select: none;
  }
</style>
</head>
<body>
<header>DES TANA’S</header>
<main>
  <div class="theme" id="themeDuJour">Chargement du thème...</div>

  <form id="uploadForm">
    <label for="fileInput">Upload ta création (image ou audio)</label>
    <input type="file" id="fileInput" accept="image/*,audio/*" required />

    <label for="texteInput">Paroles / description (pour les sons ou dessins)</label>
    <textarea id="texteInput" placeholder="Écris tes paroles ou décris ta création..." required></textarea>

    <button type="submit">Envoyer ma création</button>
  </form>

  <section class="participations" id="participations"></section>

  <div class="winner" id="winnerAnnonce"></div>
</main>
<footer>© 2025 Des Tana’s - Concours créatifs privés</footer>

<script>
  // Thèmes possibles
  const themes = [
    "Dessine une créature bizarre",
    "Outfit d’un film célèbre",
    "Écris les paroles d’un son trap",
    "Crée un personnage futuriste",
    "Imagine un tatouage original"
  ];

  // Choisir le thème du jour (jour du mois modulo thèmes)
  function themeDuJour() {
    const today = new Date();
    return themes[today.getDate() % themes.length];
  }

  // Score texte = longueur du texte
  function scoreTexte(texte) {
    return texte.trim().length;
  }

  // Score image/audio = 5 points si fichier uploadé
  function scoreFichier(aUpload) {
    return aUpload ? 5 : 0;
  }

  // Vote IA simple
  function voteIA(participations) {
    if (participations.length === 0) return null;
    let maxScore = -1;
    let winners = [];
    participations.forEach((p, i) => {
      const score = scoreTexte(p.texte) + scoreFichier(p.fichier != null);
      if (score > maxScore) {
        maxScore = score;
        winners = [i];
      } else if (score === maxScore) {
        winners.push(i);
      }
    });
    return winners[Math.floor(Math.random() * winners.length)];
  }

  // Affichage du thème
  const themeDiv = document.getElementById('themeDuJour');
  themeDiv.textContent = "Thème du jour : " + themeDuJour();

  // Gestion des participations en mémoire (localStorage)
  let participations = JSON.parse(localStorage.getItem('participations')) || [];

  // Fonction pour afficher les participations
  function afficherParticipations() {
    const cont = document.getElementById('participations');
    cont.innerHTML = "";
    participations.forEach((p, i) => {
      const div = document.createElement('div');
      div.className = 'post';
      if(p.url) {
        if(p.type.startsWith('image')) {
          const img = document.createElement('img');
          img.src = p.url;
          img.alt = "Création image";
          div.appendChild(img);
        } else if(p.type.startsWith('audio')) {
          const audio = document.createElement('audio');
          audio.controls = true;
          audio.src = p.url;
          div.appendChild(audio);
        }
      }
      const pText = document.createElement('p');
      pText.textContent = p.texte;
      div.appendChild(pText);
      cont.appendChild(div);
    });

    // Afficher le gagnant
    const winnerIndex = voteIA(participations);
    const winnerDiv = document.getElementById('winnerAnnonce');
    if(winnerIndex !== null) {
      winnerDiv.textContent = "🔥 Gagnant du jour : création n°" + (winnerIndex + 1);
    } else {
      winnerDiv.textContent = "";
    }
  }

  afficherParticipations();

  // Gestion du formulaire upload
  document.getElementById('uploadForm').addEventListener('submit', e => {
    e.preventDefault();

    const fileInput = document.getElementById('fileInput');
    const texteInput = document.getElementById('texteInput');

    if(fileInput.files.length === 0) {
      alert("Tu dois uploader un fichier !");
      return;
    }

    const fichier = fileInput.files[0];
    const texte = texteInput.value.trim();
    if(texte.length === 0) {
      alert("Tu dois écrire un texte !");
      return;
    }

    // Créer URL local pour l'affichage
    const url = URL.createObjectURL(fichier);

    participations.push({
      url: url,
      type: fichier.type,
      texte: texte
    });

    // Sauvegarder dans localStorage
    localStorage.setItem('participations', JSON.stringify(participations));

    // Reset form
    fileInput.value = "";
    texteInput.value = "";

    afficherParticipations();
  });
</script>
</body>
</html>
