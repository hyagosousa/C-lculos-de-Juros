<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>Sistema de Empréstimos</title>
  <style>
    body { font-family: Arial, sans-serif; margin: 0; background: #f5f5f5; }
    .container { max-width: 600px; margin: 18px auto; background: #fff; padding: 16px; border-radius: 10px; box-shadow: 0 0 10px rgba(0,0,0,0.08); }
    h2,h3 { text-align:center; margin: 0 0 12px 0; }
    label { font-weight:700; display:block; margin-top:10px; }
    input, select, textarea { width:100%; padding:12px; margin-top:6px; border-radius:8px; border:1px solid #ccc; font-size:16px; box-sizing:border-box; }
    .resultado { background:#f0f0f0; padding:12px; border-radius:8px; margin-top:12px; }
    button { width:100%; padding:12px; margin-top:12px; border:0; border-radius:8px; background:#007bff; color:#fff; font-size:16px; cursor:pointer; }
    button.secondary { background:#c40000; margin-top:6px; }
    table { width:100%; border-collapse:collapse; margin-top:12px; display:block; overflow-x:auto; }
    th,td { border:1px solid #ddd; padding:10px; white-space:nowrap; font-size:14px; text-align:center; }
    th { background:#fafafa; font-weight:700; }
    .valor-emp { color:red; font-weight:700; }
    .valor-juros { color:blue; font-weight:700; }
    .valor-total { color:green; font-weight:700; }
    .alert { background:#ffb3b3; padding:6px 8px; border-radius:6px; display:inline-block; font-weight:700; }

    /* MOBILE tweaks */
    @media (max-width:600px) {
      .container { width:95%; padding:12px; }
      input { font-size:18px; padding:14px; }
      button { font-size:18px; padding:14px; }
      th,td { font-size:15px; padding:10px; }
    }

    /* Login overlay (centered box) */
    .login-wrap { max-width:420px; margin:80px auto; background:#fff; padding:18px; border-radius:10px; box-shadow:0 2px 14px rgba(0,0,0,0.12); }
    .login-wrap h2 { margin-bottom:12px; }
  </style>
</head>
<body>

<script>
/*
 Simple fixed login:
 Login: H07y0321
 Senha: Helo2020@
 If not logged, show login overlay (prevents main UI from showing).
*/
if (!localStorage.getItem('logado')) {
  document.body.innerHTML = `
    <div class="login-wrap">
      <h2>Entrar</h2>
      <input id="loginUser" placeholder="Login" />
      <input id="loginPass" type="password" placeholder="Senha" style="margin-top:10px;" />
      <button id="btnEntrar" style="margin-top:12px;">Entrar</button>
    </div>
  `;
  document.getElementById('btnEntrar').onclick = function(){
    const u = document.getElementById('loginUser').value.trim();
    const p = document.getElementById('loginPass').value.trim();
    if (u === 'H07y0321' && p === 'Helo2020@') {
      localStorage.setItem('logado','true');
      location.reload();
    } else {
      alert('Login ou senha incorretos!');
    }
  };
} 
</script>

<!-- MAIN UI (renders only when logged; if not logged previous script replaced body) -->
<div class="container" id="mainUI" style="display:block;">
  <h2>Sistema de Empréstimos</h2>

  <label for="nome">Nome do Cliente:</label>
  <input id="nome" type="text" />

  <label for="cpf">CPF:</label>
  <input id="cpf" type="text" />

  <label for="telefone">Telefone (somente números):</label>
  <input id="telefone" type="text" placeholder="DDI + DDD + número" />

  <label for="endereco">Endereço:</label>
  <input id="endereco" type="text" />

  <label for="valor">Valor Emprestado (R$):</label>
  <input id="valor" type="number" step="0.01" oninput="atualizarCalculo()" />

  <label for="juros">Porcentagem de Juros (%):</label>
  <input id="juros" type="number" step="0.1" oninput="atualizarCalculo()" />

  <label for="dataEmp">Data do Empréstimo:</label>
  <input id="dataEmp" type="date" />

  <label for="dataVenc">Data de Vencimento:</label>
  <input id="dataVenc" type="date" />

  <div class="resultado">
    <p><b>Total Emprestado:</b> R$ <span id="prevValor">0.00</span></p>
    <p><b>Juros (R$):</b> <span id="prevJuros">0.00</span></p>
    <p><b>Total a Receber:</b> R$ <span id="prevFinal">0.00</span></p>
  </div>

  <button onclick="salvarCliente()">Salvar Cliente</button>
</div>

<div class="container historico">
  <h3>Clientes Registrados</h3>
  <table id="tabelaClientes">
    <thead>
      <tr>
        <th>Nome</th>
        <th>Valor</th>
        <th>Juros</th>
        <th>Total</th>
        <th>Vencimento</th>
        <th>Ações</th>
      </tr>
    </thead>
    <tbody></tbody>
  </table>
</div>

<script>
/* App JS - clean and fixed */

function formatBR(num){
  return Number(num).toFixed(2).replace('.',',');
}

function atualizarCalculo(){
  const valor = parseFloat(document.getElementById('valor').value) || 0;
  const jurosPct = parseFloat(document.getElementById('juros').value) || 0;
  const vJuros = (valor * jurosPct) / 100;
  const vFinal = valor + vJuros;
  document.getElementById('prevValor').innerText = formatBR(valor);
  document.getElementById('prevJuros').innerText = formatBR(vJuros);
  document.getElementById('prevFinal').innerText = formatBR(vFinal);
}

/* load clients from localStorage */
let clientes = JSON.parse(localStorage.getItem('clientes') || '[]');
atualizarTabela();

function salvarCliente(){
  const nome = document.getElementById('nome').value.trim();
  const cpf = document.getElementById('cpf').value.trim();
  const telefone = document.getElementById('telefone').value.trim();
  const endereco = document.getElementById('endereco').value.trim();
  const valor = parseFloat(document.getElementById('valor').value) || 0;
  const jurosPct = parseFloat(document.getElementById('juros').value) || 0;
  const dataEmp = document.getElementById('dataEmp').value || '';
  const dataVenc = document.getElementById('dataVenc').value || '';

  if (!nome) { alert('Preencha o nome do cliente.'); return; }
  if (!valor || valor <= 0) { alert('Informe um valor válido.'); return; }

  const valorJuros = (valor * jurosPct) / 100;
  const valorFinal = valor + valorJuros;

  clientes.push({
    nome, cpf, telefone, endereco,
    valor, jurosPct, valorJuros, valorFinal, dataEmp, dataVenc
  });

  localStorage.setItem('clientes', JSON.stringify(clientes));
  atualizarTabela();
  alert('Cliente salvo com sucesso!');
  // opcional: limpar campos
  document.getElementById('nome').value = '';
  document.getElementById('cpf').value = '';
  document.getElementById('telefone').value = '';
  document.getElementById('endereco').value = '';
  document.getElementById('valor').value = '';
  document.getElementById('juros').value = '';
  atualizarCalculo();
}

function excluirCliente(i){
  if (!confirm('Excluir este cliente?')) return;
  clientes.splice(i,1);
  localStorage.setItem('clientes', JSON.stringify(clientes));
  atualizarTabela();
}

function cobrar(telefone){
  const telClean = (telefone || '').replace(/\D/g,'');
  const msg = encodeURIComponent('Opa, hoje vence aquela questão do empréstimo. Pode verificar, por favor?');
  if (!telClean) {
    alert('Telefone inválido para envio do WhatsApp.');
    return;
  }
  window.open(`https://wa.me/55${telClean}?text=${msg}`, '_blank');
}

function atualizarTabela(){
  const tbody = document.querySelector('#tabelaClientes tbody');
  tbody.innerHTML = '';
  const hoje = new Date();
  clientes.forEach((c, i) => {
    let atrasado = false;
    if (c.dataVenc){
      const venc = new Date(c.dataVenc + 'T00:00:00');
      if (venc <= hoje) atrasado = true;
    }
    const row = document.createElement('tr');
    if (atrasado) row.style.background = '#ffefef';
    row.innerHTML = `
      <td>${c.nome}</td>
      <td class="valor-emp">R$ ${formatBR(c.valor)}</td>
      <td class="valor-juros">R$ ${formatBR(c.valorJuros)}</td>
      <td class="valor-total">R$ ${formatBR(c.valorFinal)}</td>
      <td>${c.dataVenc || '-'}</td>
      <td>
        <button onclick="cobrar('${c.telefone}')">Cobrar</button>
        <button onclick="excluirCliente(${i})" class="secondary">Excluir</button>
      </td>
    `;
    tbody.appendChild(row);
  });
}
</script>

</body>
</html>



