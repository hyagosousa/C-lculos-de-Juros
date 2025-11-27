<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Sistema de Empréstimos Avançado — SPA</title>
<style>
    :root{
      --bg:#f5f5f5;
      --card:#fff;
      --accent:#28a745;
      --muted:#6c757d;
      --tab-border:#ddd;
    }
    *{box-sizing:border-box}
    body { font-family: Arial, sans-serif; margin: 0; background: var(--bg); color:#111; }
    .wrap { max-width: 980px; margin: 18px auto; padding: 12px; }
    .card { background: var(--card); padding: 18px; border-radius: 10px; box-shadow: 0 4px 14px rgba(0,0,0,0.06); margin-bottom: 14px; }

    /* Tabs estilo WhatsApp-like */
    .tabs { display:flex; gap:0; border-bottom: 2px solid var(--tab-border); border-radius: 10px 10px 0 0; overflow:hidden; }
    .tab {
      flex:1;
      text-align:center;
      padding:12px 16px;
      cursor:pointer;
      background: #fff;
      font-weight:700;
      color:#333;
      user-select:none;
      border-right:1px solid var(--tab-border);
    }
    .tab:last-child{ border-right: none; }
    .tab.active { background: linear-gradient(180deg,#fff,#f7f7f7); border-bottom: 3px solid var(--accent); color: var(--accent); }

    /* Conteúdo das páginas */
    section.page { display:none; padding:14px 0; }
    section.page.active { display:block; }

    label { font-weight: bold; margin-top: 10px; display:block; color:#222; }
    input, select, textarea { width:100%; padding:12px; margin-top:6px; border-radius:8px; border:1px solid #ccc; font-size:15px; }
    button { padding:10px 14px; border:none; border-radius:8px; cursor:pointer; font-weight:700; }
    button.small { padding:6px 8px; font-size:13px; border-radius:6px; }
    .btn-success { background:var(--accent); color:white; }
    .btn-warning { background:#ffc107; color:#000; }
    .btn-info { background:#17a2b8; color:#fff; }
    .btn-danger { background:#dc3545; color:#fff; }

    .resultado, .dashboard { background:#f8f9fa; padding:12px; border-radius:8px; margin-top:12px; }
    .resultado p, .dashboard p { margin:6px 0; font-weight:700; }
    .valor-emp { color: #c82333; font-weight:800; }
    .valor-juros { color: #0b5ed7; font-weight:800; }
    .valor-total { color: #198754; font-weight:800; }

    table { width:100%; border-collapse:collapse; margin-top:10px; display:block; overflow-x:auto; }
    th, td { border:1px solid #e6e6e6; padding:10px; text-align:center; white-space:nowrap; }
    th { background:#f1f1f1; cursor:pointer; position:relative; }
    tr:nth-child(even) td { background: #fcfcfc; }
    .historico-actions button{ display:block; margin:6px 0; width:100%; }

    .piscar { animation: piscarAnim 1s infinite; }
    @keyframes piscarAnim {
        0% { background-color: #ffb3b3; }
        50% { background-color: #fff; }
        100% { background-color: #ffb3b3; }
    }

    /* Responsive */
    @media(max-width:700px){
      .tabs{ font-size:14px; }
      input, select { font-size:14px; padding:10px; }
      th, td { font-size:12px; padding:8px; }
    }

    /* Login screen styles (when injected) */
    .login-box { max-width:400px;margin:auto;margin-top:80px;background:white;padding:25px;border-radius:10px;box-shadow:0 0 10px #0003; }
    .login-box h2{margin:0 0 10px 0;text-align:center}
    .top-actions { display:flex; gap:10px; margin-top:10px; flex-wrap:wrap; }
    .muted { color:var(--muted); font-size:13px; }
</style>
</head>
<body>

<script>
/* ---------- LOGIN ---------- */
/* Mantive seu login original. Se não estiver logado, substitui a body pelo formulário de login. */
if (!localStorage.getItem("logado")) {
    document.body.innerHTML = `
    <div class="login-box">
        <h2>Login</h2>
        <input id="loginUser" placeholder="Login" />
        <input id="loginPass" type="password" placeholder="Senha" style="margin-top:10px;" />
        <button id="btnEntrar" style="width:100%;padding:12px;margin-top:16px;background:#007bff;color:white;border:none;border-radius:6px;">Entrar</button>
    </div>`;
    document.getElementById("btnEntrar").onclick = () => {
        let user = document.getElementById("loginUser").value.trim();
        let pass = document.getElementById("loginPass").value.trim();
        if (user === "H07y0321" && pass === "Helo2020@") {
            localStorage.setItem("logado", "true");
            location.reload();
        } else { alert("Login ou senha incorretos!"); }
    };
    // stop here — rest of app only loads when logged
} else {
    // build the SPA (keeps your original HTML structure but wrapped in sections/tabs)
    document.write(`
    <div class="wrap">
      <div class="card">
        <div class="tabs" role="tablist">
          <div class="tab active" data-target="paginaCadastro" onclick="mostrarPagina(event)">CADASTRO</div>
          <div class="tab" data-target="paginaResumo" onclick="mostrarPagina(event)">RESUMO</div>
          <div class="tab" data-target="paginaClientes" onclick="mostrarPagina(event)">CLIENTES</div>
        </div>

        <!-- PAGE: Cadastro -->
        <section id="paginaCadastro" class="page active">
            <h2>Sistema de Empréstimos Avançado — Cadastro</h2>

            <label>Nome do Cliente:</label>
            <input type="text" id="nome">

            <label>CPF:</label>
            <input type="text" id="cpf">

            <label>Telefone:</label>
            <input type="text" id="telefone">

            <label>Endereço:</label>
            <input type="text" id="endereco">

            <label>Valor Emprestado:</label>
            <input type="number" id="valor" step="0.01" oninput="atualizarCalculo()">

            <label>Porcentagem de Juros (%):</label>
            <input type="number" id="juros" step="0.1" oninput="atualizarCalculo()">

            <label>Data do Empréstimo:</label>
            <input type="date" id="dataEmp">

            <label>Data de Vencimento:</label>
            <input type="date" id="dataVenc">

            <div class="resultado">
                <p>Valor: <span id="prevValor" class="valor-emp">R$ 0,00</span></p>
                <p>Juros: <span id="prevJuros" class="valor-juros">R$ 0,00</span></p>
                <p>Total a Receber: <span id="prevFinal" class="valor-total">R$ 0,00</span></p>
            </div>

            <div class="top-actions">
              <button class="btn-success" onclick="salvarCliente()">Salvar Cliente</button>
              <button class="btn-warning" onclick="exportarPDF()">Exportar PDF</button>
              <button class="btn-info" onclick="document.getElementById('importFile').click()">Importar JSON</button>
              <button class="btn-danger" onclick="exportarJSON()">Backup (Exportar JSON)</button>
              <input type="file" id="importFile" accept=".json" style="display:none" onchange="importarJSON(event)">
            </div>
            <p class="muted">Dica: use o botão <strong>Backup</strong> antes de fazer grandes alterações.</p>
        </section>

        <!-- PAGE: Resumo -->
        <section id="paginaResumo" class="page">
            <h3>Resumo Geral</h3>
            <div class="dashboard">
                <p>Total de Clientes: <span id="totalClientes">0</span></p>
                <p>Clientes Pagos: <span id="totalPagos">0</span></p>
                <p>Clientes Pendentes: <span id="totalPendentes">0</span></p>
                <p>Clientes Atrasados: <span id="totalAtrasados">0</span></p>
                <p>Valor Total Emprestado: <span id="totalEmpGeral" class="valor-emp">R$ 0,00</span></p>
                <p>Total em Juros: <span id="totalJurosGeral" class="valor-juros">R$ 0,00</span></p>
                <p>Total a Receber: <span id="totalReceberGeral" class="valor-total">R$ 0,00</span></p>
            </div>

            <label>Buscar Cliente (Nome ou CPF):</label>
            <input type="text" id="buscar" oninput="atualizarTabela()">

            <label>Filtrar por Status:</label>
            <select id="filtroStatus" onchange="atualizarTabela()">
                <option value="todos">Todos</option>
                <option value="pendente">Pendentes</option>
                <option value="pago">Pagos</option>
                <option value="atrasado">Atrasados</option>
            </select>

            <div style="display:flex;gap:10px;margin-top:10px;flex-wrap:wrap;">
              <button class="btn-warning" onclick="exportarPDF()">Exportar PDF</button>
              <button class="btn-info" onclick="enviarAlertaAtrasados()">📩 Enviar alerta para atrasados</button>
            </div>
        </section>

        <!-- PAGE: Clientes -->
        <section id="paginaClientes" class="page">
            <h3>Clientes Registrados</h3>
            <div class="card" style="padding:8px;">
              <table id="tabelaClientes" aria-label="Tabela de clientes">
                <thead>
                  <tr>
                    <th onclick="ordenarTabela('nome')">Nome</th>
                    <th onclick="ordenarTabela('valor')">Valor</th>
                    <th onclick="ordenarTabela('valorJuros')">Juros</th>
                    <th onclick="ordenarTabela('valorFinal')">Total</th>
                    <th onclick="ordenarTabela('dataVenc')">Vencimento</th>
                    <th>Ações</th>
                  </tr>
                </thead>
                <tbody></tbody>
              </table>
            </div>
        </section>

      </div> <!-- card -->
    </div> <!-- wrap -->
    `);
} /* end else (logged) */
</script>

<script>
/* ---------- LÓGICA (mantendo nomes/variáveis como no seu código) ---------- */

/* Carregar clientes existentes (mesma chave 'clientes') */
let clientes = JSON.parse(localStorage.getItem("clientes") || "[]");
let ordemAtual = '';

// utility: formatar moeda (compatível)
function formatarMoeda(valor){
    // garantir Number
    let v = Number(valor || 0);
    return v.toLocaleString('pt-BR', { style: 'currency', currency: 'BRL' });
}

/* Mostrar página das tabs */
function mostrarPagina(e){
    // aceitar evento vindo do click na tab
    let target = (typeof e === 'string') ? e : e.currentTarget.getAttribute('data-target');
    // if argument is string, use as id
    if(typeof e === 'string') target = e;
    document.querySelectorAll('.tab').forEach(t=>t.classList.remove('active'));
    // marcar a tab correta
    let tab = Array.from(document.querySelectorAll('.tab')).find(t => t.getAttribute('data-target') === target);
    if(tab) tab.classList.add('active');
    // mostrar sections
    document.querySelectorAll('section.page').forEach(s => s.classList.remove('active'));
    let page = document.getElementById(target);
    if(page) page.classList.add('active');
    // atualizar conteúdos
    atualizarTabela();
    calcularTotais();
}
window.mostrarPagina = mostrarPagina;

/* atualizar pré-visualização de cálculo */
function atualizarCalculo(){
    // pode haver campos não carregados se login substituiu body — checar existência
    if(!document.getElementById('valor')) return;
    let valor = parseFloat(document.getElementById('valor').value) || 0;
    let juros = parseFloat(document.getElementById('juros').value) || 0;
    let vJuros = (valor * juros)/100;
    let vFinal = valor + vJuros;
    document.getElementById('prevValor').innerText = formatarMoeda(valor);
    document.getElementById('prevJuros').innerText = formatarMoeda(vJuros);
    document.getElementById('prevFinal').innerText = formatarMoeda(vFinal);
}
window.atualizarCalculo = atualizarCalculo;

/* salvar cliente (mantive sua função original e nomes) */
function salvarCliente(){
    // garantir elementos existam (quando usuário está logado)
    if(!document.getElementById('nome')){ alert('Erro: formulário não encontrado. Recarregue a página.'); return; }

    let nome = document.getElementById("nome").value.trim();
    let cpf = document.getElementById("cpf").value.trim();
    let telefone = document.getElementById("telefone").value.trim();
    let endereco = document.getElementById("endereco").value.trim();
    let valor = parseFloat(document.getElementById("valor").value) || 0;
    let juros = parseFloat(document.getElementById("juros").value) || 0;
    let dataEmp = document.getElementById("dataEmp").value;
    let dataVenc = document.getElementById("dataVenc").value;
    if(!nome || !cpf || !telefone || valor <= 0 || !dataEmp || !dataVenc){ alert('Preencha todos os campos corretamente!'); return; }
    let valorJuros = (valor * juros)/100;
    let valorFinal = valor + valorJuros;
    // push mantendo estrutura idêntica
    clientes.push({nome, cpf, telefone, endereco, valor, juros, valorJuros, valorFinal, dataEmp, dataVenc, pago:false});
    localStorage.setItem("clientes", JSON.stringify(clientes));
    atualizarTabela(); calcularTotais(); alert("Cliente salvo com sucesso!");

    // limpar formulário
    document.getElementById("nome").value = "";
    document.getElementById("cpf").value = "";
    document.getElementById("telefone").value = "";
    document.getElementById("endereco").value = "";
    document.getElementById("valor").value = "";
    document.getElementById("juros").value = "";
    document.getElementById("dataEmp").value = "";
    document.getElementById("dataVenc").value = "";
    atualizarCalculo();

    // ir para aba clientes automaticamente
    mostrarPagina('paginaClientes');
}
window.salvarCliente = salvarCliente;

/* excluir cliente */
function excluirCliente(i){
    if(!confirm('Confirma exclusão deste cliente?')) return;
    clientes.splice(i,1);
    localStorage.setItem("clientes", JSON.stringify(clientes));
    atualizarTabela(); calcularTotais();
}
window.excluirCliente = excluirCliente;

/* marcar pago */
function marcarPago(i){
    clientes[i].pago = true;
    localStorage.setItem("clientes", JSON.stringify(clientes));
    atualizarTabela(); calcularTotais();
}
window.marcarPago = marcarPago;

/* cobrar via WhatsApp (mantida) */
function cobrar(telefone, valorFinal, dataVenc, nome){
    // limpar telefone
    let tel = String(telefone || '').replace(/\D/g,'');
    if(!tel){ alert('Telefone inválido para envio via WhatsApp'); return; }
    let valorFormatado = Number(valorFinal || 0).toLocaleString('pt-BR', { style: 'currency', currency: 'BRL' });
    let msg = `Opa ${nome}, sua dívida de ${valorFormatado} vence em ${dataVenc}. Por favor, realize o pagamento via PIX (chave: Hyagosousasous@gmail.com). Obrigado!`;
    window.open(`https://wa.me/55${tel}?text=${encodeURIComponent(msg)}`, "_blank");
}
window.cobrar = cobrar;

/* enviar alerta para atrasados (mantida) */
function enviarAlertaAtrasados(){
    let hoje = new Date();
    let atrasados = clientes.filter(c => !c.pago && c.dataVenc && new Date(c.dataVenc) < hoje);
    if(atrasados.length === 0){ alert("Não há clientes atrasados no momento!"); return; }
    if(!confirm(`Encontrados ${atrasados.length} cliente(s) atrasado(s). Deseja abrir as cobranças no WhatsApp?`)) return;
    atrasados.forEach(c => { cobrar(c.telefone, c.valorFinal, c.dataVenc, c.nome); });
}
window.enviarAlertaAtrasados = enviarAlertaAtrasados;

/* calcular totais (mantida lógica, mas agora soma todos, mesmo pagos, para total emprestado) */
function calcularTotais(){
    let totalEmp=0, totalJuros=0, totalFinal=0;
    let pagos=0, pendentes=0, atrasados=0;
    let hoje = new Date();
    clientes.forEach(c=>{
        totalEmp += Number(c.valor || 0);
        totalJuros += Number(c.valorJuros || 0);
        totalFinal += Number(c.valorFinal || 0);
        if(c.pago) pagos++;
        else {
            pendentes++;
            if(c.dataVenc && new Date(c.dataVenc) < hoje) atrasados++;
        }
    });
    // atualizar DOM se existirem os elementos
    if(document.getElementById("totalEmpGeral")) document.getElementById("totalEmpGeral").innerText = formatarMoeda(totalEmp);
    if(document.getElementById("totalJurosGeral")) document.getElementById("totalJurosGeral").innerText = formatarMoeda(totalJuros);
    if(document.getElementById("totalReceberGeral")) document.getElementById("totalReceberGeral").innerText = formatarMoeda(totalFinal);
    if(document.getElementById("totalClientes")) document.getElementById("totalClientes").innerText = clientes.length;
    if(document.getElementById("totalPagos")) document.getElementById("totalPagos").innerText = pagos;
    if(document.getElementById("totalPendentes")) document.getElementById("totalPendentes").innerText = pendentes;
    if(document.getElementById("totalAtrasados")) document.getElementById("totalAtrasados").innerText = atrasados;
}
window.calcularTotais = calcularTotais;

/* atualizar tabela (mantida, com busca e filtros) */
function atualizarTabela(){
    let tbody = document.querySelector("#tabelaClientes tbody");
    if(!tbody) return;
    tbody.innerHTML = "";
    // valores da busca/filtro (proteger caso não existam no DOM)
    let busca = document.getElementById('buscar') ? document.getElementById('buscar').value.toLowerCase() : '';
    let filtro = document.getElementById('filtroStatus') ? document.getElementById('filtroStatus').value : 'todos';
    clientes.forEach((c,i)=>{
        let atrasado=false;
        let hoje=new Date();
        if(c.dataVenc && new Date(c.dataVenc) < hoje && !c.pago) atrasado=true;
        let corLinha = c.pago ? '#c8f7c5' : (atrasado ? '#ffb3b3' : 'white');
        // busca
        let cpfLower = String(c.cpf || '').toLowerCase();
        if(busca && !(String(c.nome || '').toLowerCase().includes(busca) || cpfLower.includes(busca))) return;
        // filtro
        if(filtro==='pendente' && (c.pago || atrasado)) return;
        if(filtro==='pago' && !c.pago) return;
        if(filtro==='atrasado' && !atrasado) return;
        // montar linha
        let tr = document.createElement('tr');
        tr.style.backgroundColor = corLinha;
        if(atrasado && !c.pago) tr.classList.add('piscar');
        // escapar apóstrofos no nome para onclick inline
        let nomeEsc = (c.nome || '').replace(/'/g,"\\'");
        tr.innerHTML = `
            <td>${c.nome}</td>
            <td class="valor-emp">${formatarMoeda(c.valor)}</td>
            <td class="valor-juros">${formatarMoeda(c.valorJuros)}</td>
            <td class="valor-total">${formatarMoeda(c.valorFinal)}</td>
            <td>${c.dataVenc || '-'}</td>
            <td class="historico-actions">
                <button class="small" onclick="cobrar('${c.telefone}', ${c.valorFinal}, '${c.dataVenc}', '${nomeEsc}')">💰 Cobrar</button>
                <button class="small btn-danger" onclick="excluirCliente(${i})">❌ Excluir</button>
                ${!c.pago ? `<button class="small btn-success" onclick="marcarPago(${i})">✅ Pago</button>` : `<button class="small" disabled>Pago</button>`}
            </td>
        `;
        tbody.appendChild(tr);
    });
}
window.atualizarTabela = atualizarTabela;

/* ordenação (mantida, agora tratando números/datas) */
function ordenarTabela(campo){
    if(ordemAtual === campo){
        clientes.reverse();
    } else {
        clientes.sort((a,b)=>{
            // números
            if(['valor','valorJuros','valorFinal'].includes(campo)){
                return (Number(a[campo]||0) - Number(b[campo]||0));
            }
            // datas
            if(campo === 'dataVenc'){
                let da = a.dataVenc ? new Date(a.dataVenc).getTime() : 0;
                let db = b.dataVenc ? new Date(b.dataVenc).getTime() : 0;
                return da - db;
            }
            // strings
            let aa = String(a[campo] || '').toLowerCase();
            let bb = String(b[campo] || '').toLowerCase();
            if(aa < bb) return -1;
            if(aa > bb) return 1;
            return 0;
        });
        ordemAtual = campo;
    }
    atualizarTabela();
}
window.ordenarTabela = ordenarTabela;

/* exportar PDF — print da área de clientes (mantida) */
function exportarPDF(){
    // preferimos imprimir a área de clientes para relatório
    let conteudo = document.getElementById('paginaClientes') ? document.getElementById('paginaClientes').innerHTML : document.body.innerHTML;
    let win = window.open('', '', 'width=900,height=700');
    win.document.write('<html><head><title>Relatório de Clientes</title></head><body style="font-family:Arial;padding:10px;">'+conteudo+'</body></html>');
    win.document.close();
    win.print();
}
window.exportarPDF = exportarPDF;

/* ---------- NOVAS FUNÇÕES: backup export/import JSON ---------- */

/* Exportar JSON (baixar arquivo com conteúdo do localStorage 'clientes') */
function exportarJSON(){
    try {
        const data = JSON.stringify(clientes, null, 2);
        const blob = new Blob([data], {type: "application/json;charset=utf-8"});
        const url = URL.createObjectURL(blob);
        const a = document.createElement('a');
        a.href = url;
        const now = new Date();
        const name = `backup_clientes_${now.toISOString().slice(0,19).replace(/:/g,'-')}.json`;
        a.download = name;
        document.body.appendChild(a);
        a.click();
        a.remove();
        URL.revokeObjectURL(url);
    } catch(e){
        alert('Erro ao exportar backup: ' + e.message);
    }
}
window.exportarJSON = exportarJSON;

/* Importar JSON: ler arquivo e mesclar (ou substituir) — mantendo clientes existentes como pedido */
function importarJSON(event){
    const file = event.target.files[0];
    if(!file) return;
    const reader = new FileReader();
    reader.onload = function(e){
        try {
            const imported = JSON.parse(e.target.result);
            if(!Array.isArray(imported)){ alert('Arquivo inválido: esperado um array de clientes'); return; }
            // Perguntar ao usuário se deseja MESCLAR ou SUBSTITUIR
            let escolha = prompt('Digite M para mesclar (adiciona os clientes do arquivo) ou S para substituir (apaga os clientes atuais). M/S', 'M');
            if(!escolha) return;
            escolha = escolha.toUpperCase();
            if(escolha === 'S'){
                clientes = imported;
                localStorage.setItem('clientes', JSON.stringify(clientes));
                alert('Clientes substituídos pelo arquivo importado.');
            } else {
                // mesclar: evitar duplicar clientes iguais (baseado em cpf + telefone)
                const existKey = (c) => `${String(c.cpf||'').trim()}|${String(c.telefone||'').trim()}`;
                const mapExist = new Map(clientes.map(c => [existKey(c), c]));
                let added = 0;
                imported.forEach(ic => {
                    const key = existKey(ic);
                    if(!mapExist.has(key)){
                        clientes.push(ic);
                        added++;
                    }
                });
                localStorage.setItem('clientes', JSON.stringify(clientes));
                alert(`Importação concluída. ${added} registro(s) adicionados (duplicados ignorados).`);
            }
            atualizarTabela(); calcularTotais();
        } catch(err){
            alert('Erro ao processar arquivo: ' + err.message);
        } finally {
            // limpar input file
            event.target.value = '';
        }
    };
    reader.readAsText(file, 'UTF-8');
}
window.importarJSON = importarJSON;

/* Inicialização: preencher tabela e totais ao carregar a página (se logged) */
document.addEventListener('DOMContentLoaded', function(){
    // se login substituiu body, as sections já foram colocadas via document.write e DOMContentLoaded pode não aplicar; chamamos direto
    atualizarTabela();
    calcularTotais();
});

// Também chamar agora para garantir que data apareça sem precisar esperar DOMContentLoaded
setTimeout(()=>{ atualizarTabela(); calcularTotais(); }, 120);

</script>

</body>
</html>
