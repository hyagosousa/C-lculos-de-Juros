<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1.0" />
<title>Sistema de Empréstimos</title>
<style>
  body { font-family: Arial, sans-serif; padding: 20px; }
  .campo { margin-bottom: 10px; }
  .vermelho { color: red; font-weight: bold; }
  .azul { color: blue; font-weight: bold; }
  .verde { color: green; font-weight: bold; }
  table { width: 100%; border-collapse: collapse; margin-top: 20px; }
  table, th, td { border: 1px solid #ccc; }
  th, td { padding: 8px; text-align: left; }
  .btn { padding: 8px 12px; margin: 5px 0; cursor: pointer; }
  .alerta { background: red; color: #fff; padding: 3px 6px; border-radius: 4px; font-size: 12px; }
</style>
</head>
<body>
<h2>Sistema de Controle de Empréstimos</h2>

<div class="campo"><input type="text" id="nome" placeholder="Nome do cliente"></div>
<div class="campo"><input type="text" id="cpf" placeholder="CPF"></div>
<div class="campo"><input type="text" id="telefone" placeholder="Telefone"></div>
<div class="campo"><input type="text" id="endereco" placeholder="Endereço"></div>
<div class="campo"><input type="number" id="valor" placeholder="Valor emprestado" class="vermelho"></div>
<div class="campo"><input type="number" id="juros" placeholder="Juros" class="azul"></div>
<div class="campo"><input type="date" id="vencimento"></div>
<button class="btn" onclick="adicionarCliente()">Adicionar Cliente</button>

<table id="tabela">
  <thead>
    <tr>
      <th>Nome</th>
      <th>CPF</th>
      <th>Telefone</th>
      <th>Endereço</th>
      <th class="vermelho">Valor</th>
      <th class="azul">Juros</th>
      <th class="verde">Total</th>
      <th>Vencimento</th>
      <th>Ações</th>
    </tr>
  </thead>
  <tbody></tbody>
</table>

<script>
let clientes = JSON.parse(localStorage.getItem("clientes")) || [];
renderTabela();

function adicionarCliente() {
  const nome = document.getElementById("nome").value;
  const cpf = document.getElementById("cpf").value;
  const telefone = document.getElementById("telefone").value;
  const endereco = document.getElementById("endereco").value;
  const valor = Number(document.getElementById("valor").value);
  const juros = Number(document.getElementById("juros").value);
  const vencimento = document.getElementById("vencimento").value;

  if (!nome || !valor || !vencimento) return alert("Preencha os campos obrigatórios");

  const total = valor + juros;

  clientes.push({ nome, cpf, telefone, endereco, valor, juros, total, vencimento });
  salvar();
  renderTabela();
}

function salvar() {
  localStorage.setItem("clientes", JSON.stringify(clientes));
}

function excluirCliente(i) {
  clientes.splice(i, 1);
  salvar();
  renderTabela();
}

function enviarCobranca(nome, telefone, total) {
  const msg = `Opa, hoje vence aquela questão. Total: R$ ${total}.`; 
  const link = `https://wa.me/55${telefone.replace(/\D/g, "")}?text=${encodeURIComponent(msg)}`;
  window.open(link, "_blank");
}

function renderTabela() {
  const tbody = document.querySelector("#tabela tbody");
  tbody.innerHTML = "";

  const hoje = new Date().toISOString().split("T")[0];

  clientes.forEach((c, i) => {
    const alerta = c.vencimento === hoje ? '<span class="alerta">VENCE HOJE</span>' : "";

    tbody.innerHTML += `
      <tr>
        <td>${c.nome}</td>
        <td>${c.cpf}</td>
        <td>${c.telefone}</td>
        <td>${c.endereco}</td>
        <td class="vermelho">R$ ${c.valor}</td>
        <td class="azul">R$ ${c.juros}</td>
        <td class="verde">R$ ${c.total}</td>
        <td>${c.vencimento} ${alerta}</td>
        <td>
          <button onclick="enviarCobranca('${c.nome}', '${c.telefone}', '${c.total}')">Cobrar</button><br>
          <button onclick="excluirCliente(${i})">Excluir</button>
        </td>
      </tr>
    `;
  });
}
</script>

<script src="https://www.gstatic.com/firebasejs/10.12.2/firebase-app-compat.js"></script>
<script src="https://www.gstatic.com/firebasejs/10.12.2/firebase-auth-compat.js"></script>
<script src="https://www.gstatic.com/firebasejs/10.12.2/firebase-firestore-compat.js"></script>
<script>
// TODO: substitua pelos seus dados do Firebase
const firebaseConfig = {
  apiKey: "SUA_API_KEY",
  authDomain: "SEU_AUTH_DOMAIN",
  projectId: "SEU_PROJECT_ID",
  storageBucket: "SEU_STORAGE_BUCKET",
  messagingSenderId: "SEU_SENDER_ID",
  appId: "SEU_APP_ID"
};
firebase.initializeApp(firebaseConfig);
const auth = firebase.auth();
const db = firebase.firestore();

// Tela de login simples (login fixo solicitado)
document.body.innerHTML = `
<div style='max-width:400px;margin:auto;margin-top:60px;background:white;padding:20px;border-radius:10px;box-shadow:0 0 10px #0002;'>
<h2>Login</h2>
<input id='email' placeholder='Login' style='width:100%;padding:10px;margin-top:10px;'>
<input id='senha' type='password' placeholder='Senha' style='width:100%;padding:10px;margin-top:10px;'>
<button id='btnLogin' style='width:100%;padding:12px;margin-top:15px;background:#007bff;color:white;border:none;border-radius:5px;'>Entrar</button>
</div>`;

btnLogin.onclick = () => {
  let usuario = document.getElementById("email").value;
  let senha = document.getElementById("senha").value;

  if (usuario === "H07y0321" && senha === "Helo2020@") {
      localStorage.setItem("logado", true);
      location.reload();
  } else {
      alert("Login ou senha incorretos!");
  }
};

// Quando logar, carrega o sistema
if (localStorage.getItem("logado")) {
    console.log("Logado com sucesso!");
}(user => {
  if (user) {
    // Aqui futuramente carregaremos a interface completa
    console.log("Logado como", user.email);
  }
});
</script>
</body>
</html>


