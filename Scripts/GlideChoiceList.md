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

```



## Best Practice 

A forma elegante e prática para capturar "Choices" de um campo de uma determinada tabela, é utilizando uma API que a Servicenow 
deixou escondida em meio a seus códigos...sério eu encontrei um tal de **GlideChoiceList** Object por um acaso analisando um **Script include** padrão, 
fiquei curioso para qual propósito servia esse tal Glide, foi ai que descobri essa belezinha


```Javascript

var glideChoices = new GlideChoiceList.getChoiceList('incident', 'state');
var choiceList = [];
for (var i=0; i < glideChoices.getSize(); i++) {
  var choice = {};
  choice.value = glideChoices.getChoice(i).getValue();
  choice.label = glideChoices.getChoice(i).getLabel();
  choiceList.push(choice);
}

```

Ok, parece mesmo ser o ideal utilizar Glide API Object's da própria Servicenow para retornar um determinado dado especifico, mas não parece nada 
elegante usar um laço como um **for** iterando até um método que retorna o tamanho do objeto **(getSize())**... dai depois preciso usar outros métodos
como **getChoices()** para retornar um valor ou label...método dentro de método, e o curioso é que a dona Servicenow não deixou uma documentação
sobre esse tesouro
