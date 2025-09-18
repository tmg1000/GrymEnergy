<!DOCTYPE html>
<html lang="sv">
<head>
  <meta charset="UTF-8">
  <title>Top Scoring SHL Players</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      padding: 20px;
      background: #f4f4f4;
    }
    .player-card {
      background: white;
      border: 1px solid #ddd;
      border-radius: 8px;
      padding: 10px;
      margin: 10px;
      width: 250px;
      box-shadow: 2px 2px 5px rgba(0,0,0,0.1);
    }
    .player-container {
      display: flex;
      flex-wrap: wrap;
    }
    .player-card h2 {
      margin: 0 0 5px;
      font-size: 1.2em;
    }
    .player-card p {
      margin: 3px 0;
    }
  </style>
</head>
<body>
  <h1>SHL Top Scoring Players</h1>
  <div id="players" class="player-container">
    <!-- Cards will be inserted here -->
  </div>

  <script>
    // Example URL: you need to replace this with the real SHL data endpoint
    const DATA_URL = 'https://fantasy.shl.se/home';

    async function fetchTopScorers() {
      try {
        const response = await fetch(DATA_URL);
        if (!response.ok) {
          throw new Error('Network response was not ok');
        }
        const data = await response.json();
        return data.players; // assuming JSON structure { players: [ { name, team, points, goals, assists, etc. } ] }
      } catch (error) {
        console.error('Error fetching top scorers:', error);
        return [];
      }
    }

    function createPlayerCard(player, rank) {
      const card = document.createElement('div');
      card.className = 'player-card';
      // rank + name
      const title = document.createElement('h2');
      title.textContent = `${rank}. ${player.name}`;
      card.appendChild(title);

      // Team
      const teamP = document.createElement('p');
      teamP.textContent = `Team: ${player.team}`;
      card.appendChild(teamP);

      // Points etc.
      const pointsP = document.createElement('p');
      pointsP.textContent = `Points: ${player.points}`;
      card.appendChild(pointsP);

      const goalsP = document.createElement('p');
      goalsP.textContent = `Goals: ${player.goals}`;
      card.appendChild(goalsP);

      const assistsP = document.createElement('p');
      assistsP.textContent = `Assists: ${player.assists}`;
      card.appendChild(assistsP);

      return card;
    }

    async function displayTopPlayers() {
      const players = await fetchTopScorers();
      const container = document.getElementById('players');
      container.innerHTML = ''; // clear

      // sort by points descending
      players.sort((a, b) => b.points - a.points);

      // display top 10
      const top = players.slice(0, 10);
      top.forEach((player, idx) => {
        const card = createPlayerCard(player, idx + 1);
        container.appendChild(card);
      });
    }

    // Run on load
    window.addEventListener('DOMContentLoaded', displayTopPlayers);
  </script>
</body>
</html>
