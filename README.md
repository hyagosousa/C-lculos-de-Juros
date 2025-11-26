<!DOCTYPE html>
<html lang="pt-br">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Controle de Empréstimos</title>
<style>
    body { font-family: Arial, sans-serif; margin: 20px; background: #f5f5f5; }
    .container { max-width: 500px; margin: auto; background: #fff; padding: 20px; border-radius: 10px; box-shadow: 0 0 10px rgba(0,0,0,0.1); }
    label { font-weight: bold; margin-top: 10px; display: block; }
    input { width: 100%; padding: 10px; margin-top: 5px; border-radius: 5px; border: 1px solid #ccc; }
    .resultado { background: #eee; padding: 10px; border-radius: 5px; margin-top: 10px; }
    button { width: 100%; padding: 12px; background: #007bff; color: white; border: none; border-radius: 5px; margin-top: 15px; font-size: 16px; cursor: pointer; }
    button:hover { background: #005fcc; }
</style>
</head>
<body>
<div class="container">
<h2>Controle de Empréstimos</h2>

<label>Nome do Cliente:</label>
<input type="text" id="nome">

<label>Valor Emprestado (R$):</label>
<input type="number" id="valor" step="0.01">

<label>Porcentagem de Juros (%):</label>
<input type="number" id="juros" step="0.1">

<label>Data do Empréstimo:</label>
<input type="date" id="dataEmp">

<label>Data de Vencimento:</label>
<input type="date" id="dataVenc">

<button onclick="calcular()">Calcular</button>

<div class="resultado">
<p><strong>Valor Emprestado:</strong> R$ <span id="rValor">0,00</span></p>
<p><strong>Valor do Juros:</strong> R$ <span id="rJuros">0,00</span></p>
<p><strong>Total a Receber:</strong> R$ <span id="rTotal">0,00</span></p>
<p><strong>Cliente:</strong> <span id="rNome">-</span></p>
<p><strong>Empréstimo:</strong> <span id="rDataEmp">-</span></p>
<p><strong>Vencimento:</strong> <span id="rDataVenc">-</span></p>
</div>
</div>

<script>
function calcular() {
    let nome = document.getElementById("nome").value;
    let valor = parseFloat(document.getElementById("valor").value);
    let juros = parseFloat(document.getElementById("juros").value);
    let dataEmp = document.getElementById("dataEmp").value;
    let dataVenc = document.getElementById("dataVenc").value;

    if (isNaN(valor) || isNaN(juros)) {
        alert("Preencha todos os valores corretamente!");
        return;
    }

    let valorJuros = valor * (juros / 100);
    let total = valor + valorJuros;

    document.getElementById("rValor").innerText = valor.toFixed(2);
    document.getElementById("rJuros").innerText = valorJuros.toFixed(2);
    document.getElementById("rTotal").innerText = total.toFixed(2);

    document.getElementById("rNome").innerText = nome;
    document.getElementById("rDataEmp").innerText = dataEmp;
    document.getElementById("rDataVenc").innerText = dataVenc;
}
</script>
</body>
</html>
