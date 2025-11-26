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
    .historico { margin-top: 20px; }
    table { width: 100%; border-collapse: collapse; display: block; overflow-x: auto; }
    th, td { border: 1px solid #ccc; padding: 10px; text-align: center; white-space: nowrap; }
    th { background: #ddd; }
    .valor-emp { color: red; font-weight: bold; }
    .valor-juros { color: blue; font-weight: bold; }
    .valor-total { color: green; font-weight: bold; }
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
        <p>Total Emprestado: R$ <span id="prevValor">0.00</span></p>
        <p>Juros (R$): <span id="prevJuros">0.00</span></p>
        <p>Total a Receber: R$ <span id="prevFinal">0.00</span></p>
    </div>

    <button onclick="salvarCliente()">Salvar Cliente</button>
</div>

<div class="container resultado">
    <h3>Resumo Geral</h3>
    <p>Total Emprestado Geral: R$ <span id="totalEmpGeral">0.00</span></p>
    <p>Total em Juros Geral: R$ <span id="totalJurosGeral">0.00</span></p>
    <p>Total a Receber Geral: R$ <span id="totalReceberGeral">0.00</span></p>
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

function atualizarCalculo(){
    let valor = parseFloat(document.getElementById('valor').value) || 0;
    let juros = parseFloat(document.getElementById('juros').value) || 0;
    let vJuros = (valor * juros)/100;
    let vFinal = valor + vJuros;
    document.getElementById('prevValor').innerText = valor.toFixed(2);
    document.getElementById('prevJuros').innerText = vJuros.toFixed(2);
    document.getElementById('prevFinal').innerText = vFinal.toFixed(2);
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

    clientes.push({nome, cpf, telefone, endereco, valor, juros, valorJuros, valorFinal, dataEmp, dataVenc});
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

function cobrar(telefone){
    let msg = encodeURIComponent("Opa, hoje vence aquela questão");
    window.open(`https://wa.me/55${telefone}?text=${msg}`, "_blank");
}

function calcularTotais(){
    let totalEmp=0, totalJuros=0, totalFinal=0;
    clientes.forEach(c => { totalEmp+=c.valor; totalJuros+=c.valorJuros; totalFinal+=c.valorFinal; });
    document.getElementById("totalEmpGeral").innerText = totalEmp.toFixed(2);
    document.getElementById("totalJurosGeral").innerText = totalJuros.toFixed(2);
    document.getElementById("totalReceberGeral").innerText = totalFinal.toFixed(2);
}

function atualizarTabela(){
    let tbody=document.querySelector("#tabelaClientes tbody");
    tbody.innerHTML="";
    clientes.forEach((c,i)=>{
        let atrasado=false;
        if(c.dataVenc){
            let hoje=new Date();
            let venc=new Date(c.dataVenc);
            if(venc<=hoje) atrasado=true;
        }
        tbody.innerHTML += `
            <tr style="background:${atrasado?'#ffb3b3':'white'};">
                <td>${c.nome}</td>
                <td class="valor-emp">R$ ${c.valor.toFixed(2)}</td>
                <td class="valor-juros">R$ ${c.valorJuros.toFixed(2)}</td>
                <td class="valor-total">R$ ${c.valorFinal.toFixed(2)}</td>
                <td>${c.dataVenc||'-'}</td>
                <td>



</body>
</html>



