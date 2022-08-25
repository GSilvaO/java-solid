# Conceitos SOLID

 Abaixo será descrito como os conceitos de SOLID foram aplicados na confecção desta aplicação

## <strong>S (Single Responsibility Principle)</strong>
 A classe [Funcionario](/src/br/com/alura/rh/model/Funcionario.java) possuia um método chamado
<strong>reajustarSalario()</strong>. Apesar deste método estar relacionado a um funcionário não é
interessante deixa-lo na classe de domínio pois é um método que pode vir a mudar e podem ser acrescentadas
novas regras com certa frequência neste método, fazendo com que a classe <i>Funcionário</i> distoe de sua 
função principal, que seria a de representar um funcionário no sistema. Este método foi refatorado para 
uma nova classe de serviço chamada [ReajusteService](/src/br/com/alura/rh/service/reajuste/ReajusteService.java) 
para que a responsabilidade de reajuste salarial fique limitada a esta classe.

## <strong>O (Open-Closed Principle)</strong>
 Ao realizar a refatoração do método <strong>reajustarSalario()</strong> para a classe <i>ReajusteService</i>
foi criado o método <strong>reajustarSalarioDoFuncionario()</strong> nesta classe. Foi observado que neste método validações deveriam ser feitas, e, caso uma nova validação fosse necessária, a tendência era esta classe 
crescer indefinidamente. Para resolver este problema, foi criada uma interface chamada [ValidacaoReajuste](src/br/com/alura/rh/service/reajuste/ValidacaoReajuste.java) com um método chamado <strong>validar()</strong>, e também foram criadas duas classes de validação que implementam esta interface, a classe [ValidacaoPercentualReajuste](src/br/com/alura/rh/service/reajuste/ValidacaoPercentualReajuste.java) e a classe 
[ValidacaoPeriodicidadeEntreReajustes](src/br/com/alura/rh/service/reajuste/ValidacaoPeriodicidadeEntreReajustes.java). 
Desta forma, foi possível passar para a classe <i>ReajusteService</i> a interface <i>ValidacaoReajuste</i>, fazendo com que caso seja necessário adicionar uma nova validação, basta a classe desta validação extender a interface <i>ValidacaoReajuste</i> e implementar seu método. Assim, a classe <i>ReajusteService</i> fica aberta para mudanças e fechada para modificações.

## <strong>L (Liskov-Substitution Principle)</strong>
 Em um cenário onde uma empresa pode contratar terceirizados, foi criada a classe [Terceirizado](/src/br/com/alura/rh/model/Terceirizado.java) para representar um terceirizado. Porém, uma indagação válida é se esta classe deveria herdar da classe <i>Funcionario</i>, já que esta classe possui um método chamado <strong>promover()</strong> e outro chamado <strong>atualizarSalario()</strong>, onde não faria sentido a classe <i>Terceirizado</i> possuir estes métodos já que a empresa contratante não seria a responsável por gerenciar assuntos relacionados ao salário e ao cargo de um funcionário terceirizado. Além disso, na classe de serviço <i>ReajusteService</i>, o método de reajuste teria que ser modificado para lidar com o caso de um terceirizado ser passado de maneira polimórfica, violando, desta forma, o princípio de substituição de Liskov. A alternativa para lidar com esta situação foi, já que a classe <i>Terceirizado</i> necessitava apenas de alguns atributos em comum com a classe <i>Funcionario</i>, estes atributos foram extraídos para uma nova classe chamada [DadosPessoais](/src/br/com/alura/rh/model/DadosPessoais.java), onde foi realizada a composição desta classe nas classes de <i>Funcionario</i> e <i>Terceirizado</i>.

## <strong>I (Interface Segregation Principle)</strong>
 A interface [Reajuste](/src/br/com/alura/rh/service/tributacao/Reajuste.java) foi criada com o objetivo de conter métodos relevantes para a operação de um reajuste. A classe [Anuenio](/src/br/com/alura/rh/service/tributacao/Anuenio.java) implementa esta interface, na qual o objetivo é fazer a lógica de reajuste salarial anual. Porém, quando um funcionário é promovido, além do reajuste salarial também há uma tributação adicional em cima deste valor e esta tributação foi, como exemplo, o valor do imposto de renda. Caso a interface <i>Reajuste</i> possuísse este método de <strong>valorDoImpostoDeRenda()</strong>, a classe <i>Anuenio</i> seria obrigada a implementa-lo e, desta forma, estaria violando o princípio de segregação de interfaces, que diz que uma classe não deveria ser forçada a implementa um método que não irá utilizar. A solução para isto foi criar uma nova interface chamada [ReajusteTributavel](/src/br/com/alura/rh/service/tributacao/ReajusteTributavel.java) que herda da interface <i>Reajuste</i>. Foi também criada uma classe chamada [Promocao](/src/br/com/alura/rh/service/tributacao/Promocao.java) onde esta, por sua vez, implementa o ajuste tributável.

## <strong>D (Dependency Inversion Principle)</strong>
 No método <strong>reajustarSalarioDoFuncionario()</strong> da classe <i>ReajusteService</i> acontecem validações para garantir que o reajuste salarial de um funcionário siga certas regras de negócio. Essas validações são implementações da interface <i>ValidacaoReajuste</i>, conforme explicado acima. Um fato interessante é que a criação das instâncias destas classes não são chamadas na classe <i>ReajusteService</i> mas injetadas no construtor através da passagem de uma lista das instância das validações. Com isso, nós invertemos a responsabilidade da criação das instâncias destas classes. O método <strong>reajustarSalarioDoFuncionario()</strong> possui um loop que irá percorrer essas instâncias e chamar o método <strong>validar()</strong> de cada validação.

