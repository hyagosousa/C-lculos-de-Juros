<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8" />
<title>Sistema de Empréstimos</title>
<style>
    body { font-family: Arial, sans-serif; margin: 0; background: #f5f5f5; }
    .container { max-width: 600px; margin: auto; background: #fff; padding: 20px; border-radius: 10px; box-shadow: 0 0 10px rgba(0,0,0,0.1); }
    label { font-weight: bold; margin-top: 10px; display: block; }
    input { width: 100%; padding: 12px; margin-top: 5px; border-radius: 8px; border: 1px solid #ccc; font-size: 16px; }
    .resultado, .historico { background: #eee; padding: 15px; border-radius: 10px; margin-top: 20px; }
    button { width: 100%; padding: 14px; background: #007bff; color: white; border: none; border-radius: 8px; margin-top: 15px; font-size: 18px; cursor: pointer; }
    button:hover { background: #005fcc; }
    table { width: 100%; border-collapse: collapse; margin-top: 15px; display: block; overflow-x: auto; }
    th, td { border: 1px solid #ccc; padding: 10px; text-align: center; font-size: 16px; white-space: nowrap; }
    th { background: #ddd; }
    .valor-emp { color: red; font-weight: bold; }
    .valor-juros { color: blue; font-weight: bold; }
    .valor-total { color: green; font-weight: bold; }

    @media (max-width: 600px) {
        .container { width: 94%; padding: 15px; }
        h2, h3 { text-align: center; font-size: 22px; }
        input { padding: 14px; font-size: 18px; }
        button { font-size: 20px; padding: 16px; }
        .resultado, .historico { padding: 12px; }
        td, th { padding: 12px; font-size: 18px; }
    }
</style>
</head>
<body>
<script>
// LOGIN SIMPLES QUE SEMPRE APARECE
// LOGIN SIMPLES QUE SEMPRE APARECE
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
        } else {
            alert("Login ou senha incorretos!");
        }
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
    <input type="number" id="valor" step="0.01">

    <label>Porcentagem de Juros (%):</label>
    <input type="number" id="juros" step="0.1">

    <label>Data do Empréstimo:</label>
    <input type="date" id="dataEmp">

    <label>Data de Vencimento:</label>
    <input type="date" id="dataVenc">

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
let clientes = JSON.parse(localStorage.getItem("clientes") || "[]");
atualizarTabela();

function salvarCliente(){
    let nome = document.getElementById("nome").value;
    let cpf = document.getElementById("cpf").value;
    let telefone = document.getElementById("telefone").value;
    let endereco = document.getElementById("endereco").value;
    let valor = parseFloat(document.getElementById("valor").value);
    let juros = parseFloat(document.getElementById("juros").value);
    let dataEmp = document.getElementById("dataEmp").value;
    let dataVenc = document.getElementById("dataVenc").value;

    let valorJuros = (valor * juros) / 100;
    let valorFinal = valor + valorJuros;

    clientes.push({ nome, cpf, telefone, endereco, valor, juros, valorJuros, valorFinal, dataEmp, dataVenc });
    localStorage.setItem("clientes", JSON.stringify(clientes));

    atualizarTabela();
    alert("Cliente salvo com sucesso!");
}

function excluirCliente(i){
    clientes.splice(i, 1);
    localStorage.setItem("clientes", JSON.stringify(clientes));
    atualizarTabela();
}

function cobrar(telefone){
    let msg = encodeURIComponent("Opa, hoje vence aquela questão");
    window.open(`https://wa.me/55${telefone}?text=${msg}`, "_blank");
}

function atualizarTabela(){
    let tbody = document.querySelector("#tabelaClientes tbody");
    tbody.innerHTML = "";

    clientes.forEach((c, i) => {
        let atrasado = false;
        if(c.dataVenc){
            let hoje = new Date();
            let venc = new Date(c.dataVenc);
            if(venc <= hoje) atrasado = true;
        }

        tbody.innerHTML += `
            <tr style="background:${atrasado ? '#ffb3b3' : 'white'};">
                <td>${c.nome}</td>
                <td class="valor-emp">R$ ${c.valor.toFixed(2)}</td>
                <td class="valor-juros">R$ ${c.valorJuros.toFixed(2)}</td>
                <td class="valor-total">R$ ${c.valorFinal.toFixed(2)}</td>
                <td>${c.dataVenc || '-'}</td>
                <td>
                    <button onclick="cobrar('${c.telefone}')">Cobrar</button>
                    <button onclick="excluirCliente(${i})" style="background:red; margin-top:5px;">Excluir</button>
                </td>
            </tr>`;
    });
}
</script>

</body>
</html>

