---
tags:
  - javascript
  - JSON
---

1. функция асинхронно загружает информацию с web-сервера, при первичной загрузке так же вызывается по событию onload  в теге `<body onload=load_data()>` со значением параметра по умолчанию requestURL:
```js
async function load_data(requestURL="http://127.0.0.1:8000/invest/10000/1000/3/")
{ 
  const request = new Request(requestURL);
  const response = await fetch(request);
  const data_list = await response.json();
  convertJSONToTable(data_list)
}
```

2. преобразует JSON в таблицу html:
```js
function convertJSONToTable(jsonData) {
  let headers = Object.keys(jsonData[0]);
  let table = '<table><thead><tr>';

  headers.forEach(header => table += `<th>${header}</th>`);
  table += '</tr></thead><tbody>';

  jsonData.forEach(row => {
    table += '<tr>';
    headers.forEach(header => table += `<td>${row[header]}</td>`);
    table += '</tr>';
  });

  table += '</tbody></table>';
  document.getElementById('table-container').innerHTML = table;
}
```

```html
<html>
<head>
  <style>
    table, th, td {
    border: 1px solid;
    }
  </style>
<script>
  function calc()
  {
    const start=document.getElementById('start').value;
    const repl=document.getElementById('repl').value;
    const term=document.getElementById('term').value;
    const requestURL=`http://127.0.0.1:8000/invest/${start}/${repl}/${term}/`
    console.log(requestURL);
    load_data(requestURL)
  }
function convertJSONToTable(jsonData) {
  let headers = Object.keys(jsonData[0]);
  let table = '<table><thead><tr>';

  headers.forEach(header => table += `<th>${header}</th>`);
  table += '</tr></thead><tbody>';

  jsonData.forEach(row => {
    table += '<tr>';
    headers.forEach(header => table += `<td>${row[header]}</td>`);
    table += '</tr>';
  });

  table += '</tbody></table>';
  document.getElementById('table-container').innerHTML = table;
}
async function load_data(requestURL="http://127.0.0.1:8000/invest/10000/1000/3/")
{ 
  const request = new Request(requestURL);
  const response = await fetch(request);
  const data_list = await response.json();
  convertJSONToTable(data_list)
}
</script>
</head>
<form id="formId" onsubmit="calc()">
Start:<input id="start" type="text" name="start" size="10"> <br>
Month replenishment: <input id="repl" type="text" name="repl" size="5"><br>
Term: <input id="term" type="text" name="term" size="3">
<input type="submit" value="Submit" />
</form>
<body onload = load_data()>

<div id="table-container">
</div>
<script>
  var form = document.getElementById("formId");
  
  function submitForm(event) {
    event.preventDefault();
    calc();
  }
  form.addEventListener('submit', submitForm);
</script>
</body>
</html>
```