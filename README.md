<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Ruleta Amigo Secreto</title>
  <style>
    :root {
      --primary: #4f46e5;
      --accent: #10b981;
      --bg: #f8fafc;
      --card: #ffffff;
      --text: #1e293b;
    }
    body {
      font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
      background: var(--bg);
      color: var(--text);
      display: flex;
      justify-content: center;
      padding: 20px;
      margin: 0;
    }
    .container {
      background: var(--card);
      width: 100%;
      max-width: 450px;
      padding: 28px;
      border-radius: 20px;
      box-shadow: 0 10px 30px rgba(0,0,0,0.08);
      text-align: center;
    }
    h2 { margin-top: 0; color: var(--primary); }
    select, button {
      width: 100%;
      padding: 12px;
      border-radius: 12px;
      font-size: 16px;
      margin-top: 12px;
      box-sizing: border-box;
    }
    select {
      border: 2px solid #e2e8f0;
      background: #fff;
    }
    button {
      background: var(--primary);
      color: #fff;
      border: none;
      font-weight: bold;
      cursor: pointer;
      transition: transform 0.1s ease;
    }
    button:active { transform: scale(0.98); }
    .roulette-box {
      margin: 24px 0;
      padding: 24px 16px;
      background: #f1f5f9;
      border-radius: 16px;
      font-size: 22px;
      font-weight: bold;
      color: #64748b;
      min-height: 35px;
      display: flex;
      align-items: center;
      justify-content: center;
      border: 2px dashed #cbd5e1;
    }
    .highlight {
      color: var(--accent) !important;
      font-size: 26px !important;
      border-color: var(--accent) !important;
      background: #ecfdf5 !important;
    }
    .secret-warning {
      font-size: 13px;
      color: #dc2626;
      margin-top: 10px;
      display: none;
      font-weight: bold;
    }
    .reset-btn {
      background: #94a3b8;
      font-size: 13px;
      padding: 8px;
      margin-top: 25px;
    }
  </style>
</head>
<body>

<div class="container">
  <h2>🎁 Ruleta Amigo Secreto</h2>
  <p>Selecciona tu nombre para girar y descubrir a tu amigo:</p>

  <select id="userSelect">
    <option value="">-- Elige quién eres --</option>
  </select>

  <button id="spinBtn" onclick="girarRuleta()">¡GIRAR RULETA!</button>

  <div class="roulette-box" id="displayBox">¿Quién te tocará?</div>
  <p class="secret-warning" id="warning">🤫 ¡Memoriza a tu amigo secreto y no se lo muestres a nadie!</p>

  <button class="reset-btn" onclick="reiniciarJuego()">Reiniciar Sorteo Completo</button>
</div>

<script>
  // 6 Participantes configurados
  const PARTICIPANTES = [
    "Elizabeth",
    "Carolina",
    "Laura",
    "Carlos",
    "Ana Marin",
    "Jacob"
  ];

  let asignados = JSON.parse(localStorage.getItem("asig_amigo_secreto_6p")) || {};

  function actualizarSelector() {
    const select = document.getElementById("userSelect");
    select.innerHTML = '<option value="">-- Elige quién eres --</option>';

    PARTICIPANTES.forEach(nombre => {
      if (!asignados[nombre]) {
        const opt = document.createElement("option");
        opt.value = nombre;
        opt.textContent = nombre;
        select.appendChild(opt);
      }
    });

    if (Object.keys(asignados).length === PARTICIPANTES.length) {
      document.getElementById("displayBox").textContent = "¡Todos tienen su amigo asignado!";
      document.getElementById("spinBtn").disabled = true;
    }
  }

  function girarRuleta() {
    const usuario = document.getElementById("userSelect").value;
    const box = document.getElementById("displayBox");
    const warning = document.getElementById("warning");
    const spinBtn = document.getElementById("spinBtn");

    if (!usuario) {
      alert("Por favor selecciona tu nombre antes de girar.");
      return;
    }

    const yaAsignados = Object.values(asignados);
    const restantesPorGirar = PARTICIPANTES.filter(n => !asignados[n] && n !== usuario);
    let candidatos = PARTICIPANTES.filter(n => n !== usuario && !yaAsignados.includes(n));

    // Si solo queda una persona pendiente después de este turno, evita que esa última quede forzada a sacarse a sí misma
    if (restantesPorGirar.length === 1) {
      const ultimo = restantesPorGirar[0];
      const opcionesQueDejanValidoAlUltimo = candidatos.filter(c => c !== ultimo);
      if (opcionesQueDejanValidoAlUltimo.length > 0) {
        candidatos = opcionesQueDejanValidoAlUltimo;
      }
    }

    if (candidatos.length === 0) {
      alert("No hay opciones disponibles.");
      return;
    }

    spinBtn.disabled = true;
    box.classList.remove("highlight");
    warning.style.display = "none";

    let contador = 0;
    const interval = setInterval(() => {
      box.textContent = PARTICIPANTES[Math.floor(Math.random() * PARTICIPANTES.length)];
      contador++;

      if (contador > 20) {
        clearInterval(interval);
        
        const seleccionado = candidatos[Math.floor(Math.random() * candidatos.length)];
        asignados[usuario] = seleccionado;
        localStorage.setItem("asig_amigo_secreto_6p", JSON.stringify(asignados));

        box.textContent = "Te tocó: " + seleccionado;
        box.classList.add("highlight");
        warning.style.display = "block";
        spinBtn.disabled = false;

        actualizarSelector();
      }
    }, 90);
  }

  function reiniciarJuego() {
    if (confirm("¿Deseas reiniciar todo el sorteo para los 6 participantes?")) {
      localStorage.removeItem("asig_amigo_secreto_6p");
      location.reload();
    }
  }

  actualizarSelector();
</script>

</body>
</html>
