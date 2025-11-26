


<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Sistema de Empréstimos</title>
<style>
    body { font-family: Arial, sans-serif; margin: 0; background: #f5f5f5; }
    .container { max-width: 800px; margin: 20px auto; background: #fff; padding: 20px; border-radius: 10px; box-shadow: 0 0 10px rgba(0,0,0,0.1); }
    h2, h3 { text-align: center; }
    label { font-weight: bold; margin-top: 10px; display: block; }
    input { width: 100%; padding: 12px; margin-top: 5px; border-radius: 8px; border: 1px solid #ccc; font-size: 16px; }
    button { width: 100%; padding: 14px; background: #007bff; color: white; border: none; border-radius: 8px; margin-top: 15px; font-size: 18px; cursor: pointer; }
    button:hover { background: #005fcc; }
    .resultado { background: #eee; padding: 15px; border-radius: 10px; margin-top: 20px; }
    .resultado p { margin: 5px 0; font-weight: bold; }
    .valor-emp { color: red; font-weight: bold; }
    .valor-juros { color: blue; font-weight: bold; }
    .valor-total { color: green; font-weight: bold; }
    .historico { margin-top: 20px; }
    table { width: 100%; border-collapse: collapse; display: block; overflow-x: auto; }
    th, td { border: 1px solid #ccc; padding: 10px; text-align: center; white-space: nowrap; }
    th { background: #ddd; }
    @media(max-width: 600px){
        input { font-size: 16px; padding: 12px; }
        button { font-size: 16px; padding: 12px; }
        td, th { font-size: 14px; padding: 8px; }
    }
</style>
</head>
<body>

<script>
// LOGIN SIMPLES
if (!localStorage.getItem("logado")) {
    document.body.innerHTML = `
    <div style='max-width:400px;margin:auto;margin-top:80px;background:white;padding:25px;border-radius:10px;box-shadow:0 0 10px #0003;'>
        <h2 style='text-align:center;'>Login</h2>
        <input id='loginUser' placeholder='Login' style='width:100%;padding:12px;margin-top:10px;'>
        <input id='loginPass' type='password' placeholder='Senha' style='width:100%;padding:12px;margin-top:10px;'>
        <button id='btnEntrar' style='width:100%;padding:14px;margin-top:18px;background:#007bff;color:white;border:none;border-radius:6px;'>Entrar</button>
    </div>`;

    document.getElementById("btnEntrar").onclick = () => {
        let user = document.getElementById("loginUser").value.trim();
        let pass = document.getElementById("loginPass").value.trim();
        if (user === "H07y0321" && pass === "Helo2020@") {
            localStorage.setItem("logado", "true");
            location.reload();
        } else { alert("Login ou senha incorretos!"); }
    };
}
</script>

<div class="container">
    <h2>Sistema de Empréstimos</h2>

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

    <button onclick="salvarCliente()">Salvar Cliente</button>
</div>

<div class="container resultado">
    <h3>Resumo Geral</h3>
    <p>Valor Total Emprestado: <span id="totalEmpGeral" class="valor-emp">R$ 0,00</span></p>
    <p>Total em Juros: <span id="totalJurosGeral" class="valor-juros">R$ 0,00</span></p>
    <p>Total a Receber: <span id="totalReceberGeral" class="valor-total">R$ 0,00</span></p>
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
let clientes = JSON.parse(localStorage.getItem("clientes") || "[]");
atualizarTabela();
calcularTotais();

function formatarMoeda(valor){
    return valor.toLocaleString('pt-BR', { style: 'currency', currency: 'BRL' });
}

function atualizarCalculo(){
    let valor = parseFloat(document.getElementById('valor').value) || 0;
    let juros = parseFloat(document.getElementById('juros').value) || 0;
    let vJuros = (valor * juros)/100;
    let vFinal = valor + vJuros;
    document.getElementById('prevValor').innerText = formatarMoeda(valor);
    document.getElementById('prevJuros').innerText = formatarMoeda(vJuros);
    document.getElementById('prevFinal').innerText = formatarMoeda(vFinal);
}

function salvarCliente(){
    let nome = document.getElementById("nome").value;
    let cpf = document.getElementById("cpf").value;
    let telefone = document.getElementById("telefone").value;
    let endereco = document.getElementById("endereco").value;
    let valor = parseFloat(document.getElementById("valor").value) || 0;
    let juros = parseFloat(document.getElementById("juros").value) || 0;
    let dataEmp = document.getElementById("dataEmp").value;
    let dataVenc = document.getElementById("dataVenc").value;

    let valorJuros = (valor * juros)/100;
    let valorFinal = valor + valorJuros;

    clientes.push({nome, cpf, telefone, endereco, valor, juros, valorJuros, valorFinal, dataEmp, dataVenc, pago:false});
    localStorage.setItem("clientes", JSON.stringify(clientes));

    atualizarTabela();
    calcularTotais();
    alert("Cliente salvo com sucesso!");
}

function excluirCliente(i){
    clientes.splice(i,1);
    localStorage.setItem("clientes", JSON.stringify(clientes));
    atualizarTabela();
    calcularTotais();
}

function marcarPago(i){
    clientes[i].pago = true;
    localStorage.setItem("clientes", JSON.stringify(clientes));
    atualizarTabela();
    calcularTotais();
}

function cobrar(telefone){
    let msg = encodeURIComponent("Opa, hoje vence aquela questão");
    window.open(`https://wa.me/55${telefone}?text=${msg}`, "_blank");
}

function calcularTotais(){
    let totalEmp=0, totalJuros=0, totalFinal=0;
    clientes.forEach(c => {
        if(!c.pago){
            totalEmp+=c.valor;
            totalJuros+=c.valorJuros;
            totalFinal+=c.valorFinal;
        }
    });
    document.getElementById("totalEmpGeral").innerText = formatarMoeda(totalEmp);
    document.getElementById("totalJurosGeral").innerText = formatarMoeda(totalJuros);
    document.getElementById("totalReceberGeral").innerText = formatarMoeda(totalFinal);
}

function atualizarTabela(){
    let tbody=document.querySelector("#tabelaClientes tbody");
    tbody.innerHTML="";
    clientes.forEach((c,i)=>{
        let atrasado=false;
        if(c.dataVenc){
            let hoje=new Date();
            let venc=new Date(c.dataVenc);
            if(venc<=hoje && !c.pago) atrasado=true;
        }
        let corLinha = c.pago ? '#c8f7c5' : (atrasado ? '#ffb3b3' : 'white');
        tbody.innerHTML += `
            <tr style="background:${corLinha};">
                <td>${c.nome}</td>
                <td class="valor-emp">${formatarMoeda(c.valor)}</td>
                <td class="valor-juros">${formatarMoeda(c.valorJuros)}</td>
                <td class="valor-total">${formatarMoeda(c.valorFinal)}</td>
                <td>${c.dataVenc||'-'}</td>
                <td>
                    <button onclick="cobrar('${c.telefone}')">Cobrar</button>
                    <button onclick="excluirCliente(${i})" style="background:red; margin-top:5px;">Excluir</button>
                    ${!c.pago ? `<button onclick="marcarPago(${i})" style="background:green; margin-top:5px;">Pago</button>` : ''}
                </td>
            </tr>`;
    });
}
</script>

</body>
</html>
