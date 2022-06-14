# GlideChoiceList


Em muitos momentos em nossa jornada de escrita de scripts na plataforma, precisamos capturar um conjunto de "Choices" de um campo numa tabela, 
e é muito comum pensarmos em buscar essas informaçôes diretamente na fonte, na tabela **sys_choice** via **GlideRecord**, porém isso é muito custoso para 
o servidor e pra você, pois irá abrir uma nova conexão para para buscar os registros, sendo necessário passar os parametros de query como: 
Table, Element e Language


### Seria algo do tipo: 

```Javascript

var grSysChoice = new GlideRecord('sys_choice');
grSysChoice.addEncodedQuery("name=incident^element=state^language=pb");
grSysChoice.query();
var choiceList = []
while (grSysChoice.next()) {
    var choice = {};
    choice.label = grSysChoice.getValue('value');
    choice.value = grSysChoice.getValue('value');
    choiceList.push(choice);
}

return choiceList;

/*
    Output ()=>
        [
            {label: "New", value: "1"},
            {label: "In Progress", "value": "2"},
            {label: "On Hold", "value": "3"}
        ]
*/

```


## A melhor prática

A forma elegante e prática para capturar "Choices" de um campo de uma determinada tabela, é utilizando uma API que a Servicenow 
deixou escondida em meio a seus códigos...sério eu encontrei um tal de **GlideChoiceList** Object por um acaso analisando um **Script include** padrão, 
fiquei curioso para qual propósito servia esse tal Glide, foi ai que descobri essa belezinha


```Javascript

var glideChoices = new GlideChoiceList.getChoiceList('incident', 'state');
var choiceList = [];
for (var i=0; i < glideChoices.getSize(); i++) {
  var choice = glideChoices.getChoice(i);
  choiceList.push({
    label: choice.getLabel(),
    value: choice.getValue()
  });
}

return glideChoices

/*
    Output ()=>
        [
            {label: "New", value: "1"},
            {label: "In Progress", "value": "2"},
            {label: "On Hold", "value": "3"}
        ]
*/

```

## Vamos tentar entender...

Ok, parece mesmo ser o ideal utilizar Glide API Object's da própria Servicenow para retornar um determinado dado especifico, mas oque realmente está
acontecendo? vamos tentar entender a estrutura da chamada:


1. A primeira linha, se trata de uma instancia do objeto **GlideChoiceList** que retorna um objeto do Javascript juntamente com um método construtor
chamado **getChoiceList** passando como parametro duas Strings: Nome da Tabela, Nome do Campo


```Javascript

var glideChoices = new GlideChoiceList.getChoiceList('incident', 'state');

/*
    Output ()=>
    [Object]
*/

```


2. Logo em seguida da declaração do objeto dentro da variável **glideChoices**, temos um método GET chamado **getSize()** na qual retorna quantas choices
estão cadastradas para esse campo 


```Javascript

glideChoices.getSize()

/*
    Output ()=>
    3
*/
   
```
3. Analisando os retornos dos métodos, não consegui chegar a uma conclusão de como o Objeto GlideChoiceList é estruturado através de logs, mas entendi 
que havia uma cadeia de elementos que poderiam ser acessados através de um método chamado **getChoice()** de forma sequencial numérica, como posições de
array (0,1,2,3) percorrendo todas as choices encontradas de um campo


```Javascript

glideChoices.getChoice(0).label   

/*
    Output ()=>
    "New"
*/

```

4. Agora basta usarmos uma estrutura de repetição como um **for** baseada na quantidade de elementos retornados do método **getSize*()** e em seguida podemos criar nosso próprio objeto utilizando alguns métodos GETTERS como **getLabel()** e **getValue()** de forma segura

```Javascript

var choiceList = [];
for (var i=0; i < glideChoices.getSize(); i++) {
  choiceList.push({
    label: choice.getChoice(i).getLabel(),
    value: choice.getChoice(i).getValue()
  });)
}

/*
    Output ()=>
        [
            {label: "New", value: "1"},
            {label: "In Progress", "value": "2"},
            {label: "On Hold", "value": "3"}
        ]
*/


```


Simples assim!!! existem diversas técnicas que nos livram de criar consultas com GlideRecord por exemplo, de forma desnecessária, e para descobrir isso, estude tudo que envolve script dentro plataforma, dessa forma você descobrirá um mundo de possibilidades fora da caixinha. 
