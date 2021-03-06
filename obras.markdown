---
layout: page
title: Obras
permalink: /obras/
onLoadFunction: processURLParams()
---
<p>Obs.: As citações das obras respeitaram a gramática e ortografia da época em que foram escritas as obras.</p>
<div class="navbar">
    <input class="nameobra" id="termo" placeholder="🔍 Título da Obra" oninput="processParams()">
    <input class="nameautora" id="nomeautora" placeholder="🔍 Autora" oninput="processParams()">
    <select class="pagenum" id="opp" onchange="processParams()">
        <option value="5" disabled selected>Obras por Página</option>
        <option value="5">5</option>
        <option value="10">10</option>
        <option value="25">25</option>
        <option value="50">50</option>
        <option value="100">100</option>
        <option value="Todas">Todas</option>
    </select>
    <select class="obracat" id="filtros" onchange="processParams()">
        <option value="Todas" disabled selected>Estilo das Obras</option>
        <option value="Todas">Todas</option>
        <option value="Prosa">Prosa</option>
        <option value="Poesia">Poesia</option>
    </select>
</div>
<div style="align-items: center; text-align: center; line-height: 80px">
  <button class="button" onclick="randomObra()" style="font-weight:900; box-shadow: #00000044 0px 3px 2px; display: inline-block;">Me mostre uma obra aleatória!</button>
</div>
<div class="pagination" id="pagination-top"></div>
<p id="demo"></p>
<div class="pagination" id="pagination-bottom">
    <span class="paginationbutton" onclick="pageButton('f')" id="pgnt-btn-f"><<</span>
    <span class="paginationbutton" onclick="pageButton('a')" id="pgnt-btn-a">1</span>
    <span class="paginationbutton" onclick="pageButton('b')" id="pgnt-btn-b">2</span>
    <span class="paginationbutton" onclick="pageButton('c')" id="pgnt-btn-c">3</span>
    <span class="paginationbutton" onclick="pageButton('d')" id="pgnt-btn-d">4</span>
    <span class="paginationbutton" onclick="pageButton('e')" id="pgnt-btn-e">5</span>
    <span class="paginationbutton" onclick="pageButton('l')" id="pgnt-btn-l">>></span>
</div>
<script>
var share = true;

document.documentElement.style.setProperty('--scalefac', (document.documentElement.clientWidth / 640).toString())

var obras = [
{% for pagina in site.pages %}{% if pagina.dir == "/obras/"%}{% if pagina.name != "obras.markdown"%}
{titulo:"{{ pagina.nomelivro }}", autora:"{{ pagina.nomeautora }}", ano:"{{ pagina.anolancamento }}", escola:"{% if pagina.revista != true %}{{ pagina.layout }}{% else %}prosa{% endif %}", imagem:"{{ pagina.imagemcapa }}", link:"{{ pagina.nomelivro | slugify: "latin"}}", dest:"{{ pagina.link  | strip_newlines || default: 'https://www.youtube.com/watch?v=dQw4w9WgXcQ' }}", destname:"{{ pagina.fontelivro | default: "YouTube" }}", quote:"{{ pagina.quote | strip_newlines }}", quotepag:"{{ pagina.quotepagina }}"},
{% endif %}{% endif %}{% endfor %}{}];

delete obras[obras.length - 1];

var obrasMesmo = [];
var obrasDeVerdade = [];
var obrasPraMostrar = [];

var obrasPorPagina = 10, atualPagina = 1, pagMax = 1;

// Processamento de parâmetros no URL
function processURLParams()
{
  var url = window.location.href;
  if(url.includes("?"))
  {
    var params = url.substring(url.indexOf("?")+1).split("&");
    for(i in params)
    {
      setParam(params[i]);
    }
  }
  processParams();
}

// Método para mudar valores do HTML baseado no URL
function setParam(param)
{
  var values = param.split("=")
  if(values.length = 2)
  {
    try
    {
      document.getElementById(values[0]).value = decodeURIComponent(values[1]); 
    }
    catch(err)
    {
      
    }
  }
}

// Processamento do número de páginas
function processParams(novaPag)
{
  document.getElementById("pagination-top").innerHTML = "";
  obrasPorPagina = parseInt(document.getElementById("opp").options[document.getElementById("opp").selectedIndex].value);
  if(!Number.isInteger(obrasPorPagina))
  {
    obrasPorPagina = 99999;
  }
  if(novaPag != null && Number.isInteger(novaPag))
  {
    atualPagina = novaPag;
  }
  else
  {
    atualPagina = 1;
  }
  if(atualPagina < 1) atualPagina = 1;
  autora();
}

// Filtro por nome de autora
function autora()
{
	obrasMesmo = [];
	var autoraBar = document.getElementById("nomeautora");
	var autoraVal = autoraBar.value;
    	for(i in obras)
    	{
		var novaAutora = string_to_slug_mod(obras[i].autora);
    		if(novaAutora.includes(string_to_slug_mod(autoraVal)))
        	{
        		obrasMesmo[obrasMesmo.length] = obras[i];
        	}
    	}
	search();
}

// Filtro por nome de obra
function search()
{
	obrasDeVerdade = [];
	var searchBar = document.getElementById("termo");
	var termo = searchBar.value;
    	for(i in obrasMesmo)
    	{
		var novoTitulo = string_to_slug_mod(obrasMesmo[i].titulo);
    		if(novoTitulo.includes(string_to_slug_mod(termo)))
        	{
        		obrasDeVerdade[obrasDeVerdade.length] = obrasMesmo[i];
        	}
    	}
	tipoDeObra();
}

// Filtro por tipo de texto
function tipoDeObra() {
  obrasPraMostrar = [];
  var escolaOptions = document.getElementById("filtros");
  var escola = escolaOptions.options[escolaOptions.selectedIndex].value;
  document.getElementById("demo").innerHTML = "";

  for (i in obrasDeVerdade)
  {
    if(escola == "Todas" || obrasDeVerdade[i].escola == escola.toLowerCase())
    {
      obrasPraMostrar[obrasPraMostrar.length] = obrasDeVerdade[i];
    }
  }

  listarObras();
}

function listarObras()
{
  // Paginação
  pagMax = Math.ceil(obrasPraMostrar.length / obrasPorPagina);
  if(atualPagina > pagMax) atualPagina = pagMax;
  var obraOffset = (obrasPorPagina * (atualPagina - 1));
  var obraEmPag = 0;

  // Adição de obras no HTML
  for(val in obrasPraMostrar)
  {
    if(obraEmPag >= obrasPorPagina) continue;
    var i = parseInt(val) + obraOffset;
    if(i >= obrasPraMostrar.length) continue;
    if(obrasPraMostrar[i].titulo.toLowerCase().includes("revista"))
    {
      var buttons = "";
      for(link in obrasPraMostrar[i].dest.split(" $ "))
      {
        var linkURL = obrasPraMostrar[i].dest.split(" $ ")[link].split(" # ")[0];
        var linkEdition = obrasPraMostrar[i].dest.split(" $ ")[link].split(" # ")[1];
        buttons += '<button class="button" style="display: inline-block;" onclick=\'window.open("' + linkURL + '", "_self")\'>Acesse a ' + linkEdition + 'ª Edição via ' + obrasPraMostrar[i].destname + '!</button><br><br>\n';
      }
        document.getElementById("demo").innerHTML += 
        '<div class="bookpreview">\n'+
        '<tag style="font-weight:900;font-size:36px">' + obrasPraMostrar[i].titulo + '</tag>\n<br>\n' +
        '<tag style="color:#505050;font-size:25px"><i><b>' + obrasPraMostrar[i].autora + ' (Redatora-Chefe)</b> - ' + obrasPraMostrar[i].ano + '</i></tag>\n<br>\n' +
        '<div class="quote" style="font-weight:400;background-color: #0000;color: black;text-shadow: none;">\n<i>\n' +
        '<div style="text-align: justify">' + obrasPraMostrar[i].quote + '</div><br>\n' + '</i></div>' +
        '<div style="text-align: center">' + buttons + "</div><br>\n" +
        '<a href="https://api.whatsapp.com/send?text=Olha%20essa%20obra%20maravilhosa%20da%20' + encodeURI(obrasPraMostrar[i].autora) + '%20que%20eu%20encontrei%21%0A' + encodeURI("{{ site.url }}obras/" + obrasPraMostrar[i].link) + '" target="_blank"><img src="https://elas-na-literatura.github.io/rsc/whatsapp.svg" alt="WhatsApp" style="margin-top:-11px;margin-right:5px;"></a>'+
        '<a href="https://twitter.com/intent/tweet?hashtags=ElasNaLiteratura&original_referer=https%3A%2F%2Fpublish.twitter.com%2F&ref_src=twsrc%5Etfw&text=Olha%20essa%20obra%20maravilhosa%20da%20' + encodeURI(obrasPraMostrar[i].autora) + '%20que%20eu%20encontrei!%20&tw_p=tweetbutton&url=' + encodeURI("{{ site.url }}obras/" + obrasPraMostrar[i].link) + '" target="_blank"><img src="https://elas-na-literatura.github.io/rsc/twitter.svg" alt="Twitter" style="margin-top:-11px;margin-right:5px;"></a>'+
        '<iframe src="https://www.facebook.com/plugins/share_button.php?href=' + encodeURI("{{ site.url }}obras/" + obrasPraMostrar[i].link) + '&layout=button&size=small&width=110&height=20&appId" width="110" height="20" style="border:none;overflow:hidden" scrolling="no" frameborder="0" allowTransparency="true" allow="encrypted-media"></iframe>\n'+
        '\n</div>\n<br>\n';
        obraEmPag++;
        continue;
    }
    switch(obrasPraMostrar[i].escola)
    {
      case "prosa":
        if(obrasPraMostrar[i].imagem != "")
        {
          document.getElementById("demo").innerHTML += 
          '<div class="bookpreview">\n'+
          '<div class="row">\n'+
          '<div class="columncapatwo">\n<img src=' + obrasPraMostrar[i].imagem + '>\n</div>\n'+
          '<div class="columntwo">\n'+
          '<tag style="font-weight:900;font-size:36px">' + obrasPraMostrar[i].titulo + '</tag>\n<br>\n' +
          '<tag style="color:#505050;font-size:25px"><i><b>' + obrasPraMostrar[i].autora + '</b> - ' + obrasPraMostrar[i].ano + '</i></tag>\n<br><br>\n' +
          '<button class="button" onclick=\'window.open("{{ site.url }}obras/' + obrasPraMostrar[i].link + '", "_self")\'>Conferir Obra</button>\n</div>\n</div>\n<br>\n'+
          '<a href="https://api.whatsapp.com/send?text=Olha%20essa%20obra%20maravilhosa%20da%20' + encodeURI(obrasPraMostrar[i].autora) + '%20que%20eu%20encontrei%21%0A' + encodeURI("{{ site.url }}obras/" + obrasPraMostrar[i].link) + '" target="_blank"><img src="https://elas-na-literatura.github.io/rsc/whatsapp.svg" alt="WhatsApp" style="margin-top:-12px;margin-right:5px;"></a>'+
          '<a href="https://twitter.com/intent/tweet?hashtags=ElasNaLiteratura&original_referer=https%3A%2F%2Fpublish.twitter.com%2F&ref_src=twsrc%5Etfw&text=Olha%20essa%20obra%20maravilhosa%20da%20' + encodeURI(obrasPraMostrar[i].autora) + '%20que%20eu%20encontrei!%20&tw_p=tweetbutton&url=' + encodeURI("{{ site.url }}obras/" + obrasPraMostrar[i].link) + '" target="_blank"><img src="https://elas-na-literatura.github.io/rsc/twitter.svg" alt="Twitter" style="margin-top:-12px;margin-right:5px;"></a>'+
          '<iframe src="https://www.facebook.com/plugins/share_button.php?href=' + encodeURI("{{ site.url }}obras/" + obrasPraMostrar[i].link) + '&layout=button&size=small&width=110&height=20&appId" width="110" height="20" style="border:none;overflow:hidden" scrolling="no" frameborder="0" allowTransparency="true" allow="encrypted-media"></iframe>\n'+
          '</div>\n<br>\n';
        }
        else
        {
          document.getElementById("demo").innerHTML += 
          '<div class="bookpreview">\n'+
          '<tag style="font-weight:900;font-size:36px">' + obrasPraMostrar[i].titulo + '</tag>\n<br>\n' +
          '<tag style="color:#505050;font-size:25px"><i><b>' + obrasPraMostrar[i].autora + '</b> - ' + obrasPraMostrar[i].ano + '</i></tag>\n<br><br>\n' +
          '<button class="button" onclick=\'window.open("{{ site.url }}obras/' + obrasPraMostrar[i].link + '", "_self")\'>Conferir Obra</button>\n<br><br>\n'+
          '<a href="https://api.whatsapp.com/send?text=Olha%20essa%20obra%20maravilhosa%20da%20' + encodeURI(obrasPraMostrar[i].autora) + '%20que%20eu%20encontrei%21%0A' + encodeURI("{{ site.url }}obras/" + obrasPraMostrar[i].link) + '" target="_blank"><img src="https://elas-na-literatura.github.io/rsc/whatsapp.svg" alt="WhatsApp" style="margin-top:-12px;margin-right:5px;"></a>'+
          '<a href="https://twitter.com/intent/tweet?hashtags=ElasNaLiteratura&original_referer=https%3A%2F%2Fpublish.twitter.com%2F&ref_src=twsrc%5Etfw&text=Olha%20essa%20obra%20maravilhosa%20da%20' + encodeURI(obrasPraMostrar[i].autora) + '%20que%20eu%20encontrei!%20&tw_p=tweetbutton&url=' + encodeURI("{{ site.url }}obras/" + obrasPraMostrar[i].link) + '" target="_blank"><img src="https://elas-na-literatura.github.io/rsc/twitter.svg" alt="Twitter" style="margin-top:-12px;margin-right:5px;"></a>'+
          '<iframe src="https://www.facebook.com/plugins/share_button.php?href=' + encodeURI("{{ site.url }}obras/" + obrasPraMostrar[i].link) + '&layout=button&size=small&width=110&height=20&appId" width="110" height="20" style="border:none;overflow:hidden" scrolling="no" frameborder="0" allowTransparency="true" allow="encrypted-media"></iframe>\n'+
          '</div>\n<br>\n';
        }
        break;
      case "poesia":
        document.getElementById("demo").innerHTML += 
        '<div class="bookpreview">\n'+
        '<tag style="font-weight:900;font-size:36px">' + obrasPraMostrar[i].titulo + '</tag>\n<br>\n' +
        '<tag style="color:#505050;font-size:25px"><i><b>' + obrasPraMostrar[i].autora + '</b> - ' + obrasPraMostrar[i].ano + '</i></tag>\n<br>\n' +
        '<div class="quote" style="font-weight:400">\n<i>\n<tag style="font-size: 200%">&#x201c</tag><br>\n' +
        '<div class="center">' + obrasPraMostrar[i].quote + '</div><br>\n<div style="font-size: 200%" class="right"> &#x201d <br>\n' +
        '<p style="font-size: 50%"><tag style="font-weight:750">' + obrasPraMostrar[i].titulo + '</tag>, ' + obrasPraMostrar[i].quotepag + 'ª Estrofe.</p></div></i></div>' +
        '<button class="button" onclick=\'window.open("' + obrasPraMostrar[i].dest + '", "_self")\'>Acesse via ' + obrasPraMostrar[i].destname + '!</button><br><br>\n'+
        '<a href="https://api.whatsapp.com/send?text=Olha%20essa%20poesia%20maravilhosa%20da%20' + encodeURI(obrasPraMostrar[i].autora) + '%20que%20eu%20encontrei%21%0A' + encodeURI("{{ site.url }}obras/" + obrasPraMostrar[i].link) + '" target="_blank"><img src="https://elas-na-literatura.github.io/rsc/whatsapp.svg" alt="WhatsApp" style="margin-top:-11px;margin-right:5px;"></a>'+
        '<a href="https://twitter.com/intent/tweet?hashtags=ElasNaLiteratura&original_referer=https%3A%2F%2Fpublish.twitter.com%2F&ref_src=twsrc%5Etfw&text=Olha%20essa%20poesia%20maravilhosa%20da%20' + encodeURI(obrasPraMostrar[i].autora) + '%20que%20eu%20encontrei!%20&tw_p=tweetbutton&url=' + encodeURI("{{ site.url }}obras/" + obrasPraMostrar[i].link) + '" target="_blank"><img src="https://elas-na-literatura.github.io/rsc/twitter.svg" alt="Twitter" style="margin-top:-11px;margin-right:5px;"></a>'+
        '<iframe src="https://www.facebook.com/plugins/share_button.php?href=' + encodeURI("{{ site.url }}obras/" + obrasPraMostrar[i].link) + '&layout=button&size=small&width=110&height=20&appId" width="110" height="20" style="border:none;overflow:hidden" scrolling="no" frameborder="0" allowTransparency="true" allow="encrypted-media"></iframe>\n'+
        '\n</div>\n<br>\n';
        break;
    }
    obraEmPag++;
  }

  buildPagination();
}

function buildPagination()
{
  var currentPageButton = -1;

  if(pagMax <= 5)
  {
    document.getElementById("pgnt-btn-f").style.display = "none";
    document.getElementById("pgnt-btn-l").style.display = "none";
  }
  else
  {
    document.getElementById("pgnt-btn-f").style.display = "inline-block";
    document.getElementById("pgnt-btn-l").style.display = "inline-block";
  }


  if(atualPagina == 1)
  {
    currentPageButton = 0;
  }
  else if(atualPagina == 2)
  {
    currentPageButton = 1;
  }
  else if((pagMax > 5 && atualPagina < pagMax - 1) || (pagMax <= 5 && atualPagina == 3))
  {
    currentPageButton = 2;
  }
  else if((pagMax > 5 && atualPagina == pagMax - 1) || (pagMax <= 5 && atualPagina == 4))
  {
    currentPageButton = 3;
  }
  else if((pagMax > 5 && atualPagina == pagMax) || (pagMax <= 5 && atualPagina == 5))
  {
    currentPageButton = 4;
  }

  var buttonIndex = 0;
  while(buttonIndex < 5)
  {
    var paginaToDisplay = (atualPagina + buttonIndex - currentPageButton);
    if(paginaToDisplay <= pagMax)
    {
      document.getElementById("pgnt-btn-"+String.fromCharCode(97 + buttonIndex)).innerHTML = paginaToDisplay.toString();
      document.getElementById("pgnt-btn-"+String.fromCharCode(97 + buttonIndex)).style.display = "inline-block";
    }
    else
    {
      document.getElementById("pgnt-btn-"+String.fromCharCode(97 + buttonIndex)).style.display = "none";
    }
    buttonIndex += 1;
  }

  document.getElementById("pgnt-btn-"+String.fromCharCode(97 + currentPageButton)).innerHTML = "<b>" + atualPagina.toString() + "</b>";

  document.getElementById("pagination-top").innerHTML = document.getElementById("pagination-bottom").innerHTML;
}

function string_to_slug_mod (str) {
    str = str.replace(/^\s+|\s+$/g, ''); // trim
    str = str.toLowerCase();
    // remove accents, swap ñ for n, etc
    var from = "ãàáäâèéëêìíïîòóöôùúüûñç·/_,:;õ";
    var to   = "aaaaaeeeeiiiioooouuuunc      o";
    for (var i=0, l=from.length ; i<l ; i++) {
        str = str.replace(new RegExp(from.charAt(i), 'g'), to.charAt(i));
    }
    str = str.replace(/[^a-z0-9 -]/g, '') // remove invalid chars
        .replace(/\s+/g, ' ') // collapse whitespace and replace by spacebar
        .replace(/ +/g, ' '); // collapse spaces
	return str;
}

function randomObra()
{
  var random = Math.floor(obras.length * Math.random());
  window.open("{{  site.url }}obras/" + obras[random].link, "_self");
}

window.addEventListener("resize", function()
{
  document.documentElement.style.setProperty('--scalefac', (document.documentElement.clientWidth / 640).toString())
});

function pageButton(buttonID)
{
  var switchPage = false;
  var pageToSwitch = 0;
  switch (buttonID)
  {
    case 'f':
      switchPage = true;
      pageToSwitch = 1;
      break;
    case 'a':
      switchPage = true;
      pageToSwitch = parseInt(document.getElementById("pgnt-btn-a").innerText);
      break;
    case 'b':
      switchPage = true;
      pageToSwitch = parseInt(document.getElementById("pgnt-btn-b").innerText);
      break;
    case 'c':
      switchPage = true;
      pageToSwitch = parseInt(document.getElementById("pgnt-btn-c").innerText);
      break;
    case 'd':
      switchPage = true;
      pageToSwitch = parseInt(document.getElementById("pgnt-btn-d").innerText);
      break;
    case 'e':
      switchPage = true;
      pageToSwitch = parseInt(document.getElementById("pgnt-btn-e").innerText);
      break;
    case 'l':
      switchPage = true;
      pageToSwitch = 99999999999;
      break;
  }
  if(switchPage) { processParams(pageToSwitch); }
}

</script>
<!--<div style="position: fixed;width: 600px;bottom: 15px;margin: auto;/* min-width: 300px; */border-radius: 5px;background: #F0F0F0;border: 2px solid #CDCDCD;box-shadow: 0px 5px 10px #AAAAAA;z-index: 50;padding: 5px 5px;align-content: center;">
<form>
Página Atual
<input type="number" id="paginaatual" value="1" oninput="processParams()"><br>
</form>
</div>-->