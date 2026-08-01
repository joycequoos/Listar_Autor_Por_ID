# Interface, Service, Controller e Endpoint: Buscar Autor por ID

[← Voltar](https://github.com/joycequoos/WEB-API-com-.NET-8-e-SQL-Server)

![Fluxo Controller, Interface e Service](https://github.com/joycequoos/Controllers_Services/blob/main/img/01_Fx_Controller_Interface_Service_2.jpg)

Dando continuidade ao endpoint de listagem de autores, esta etapa implementa a busca de um autor específico pelo seu **ID**, seguindo o mesmo fluxo já estabelecido: Service → Controller → Teste do endpoint.

## 1. Implementando o Método no AutorService.cs

O método `BuscarAutorPorId`, criado como interface no repositório anterior, é implementado no `AutorService.cs`. Ele é um método assíncrono que busca, no banco de dados, um autor com base no ID informado (`idAutor`).

![Método BuscarAutorPorId no AutorService](https://github.com/joycequoos/Listar_Autor_Por_ID/blob/main/img/AutorService_BuscarID.png)

### Explicação do Código

- O método retorna um objeto do tipo `ResponseModel<AutorModel>`.
- Ele usa o Entity Framework Core para acessar `_context.Autores`, um `DbSet<AutorModel>` que representa a tabela de autores no banco de dados.
- A busca é feita com `FirstOrDefaultAsync`, que retorna o primeiro autor com o ID fornecido, ou `null` caso não seja encontrado.
- Se o autor **não** for encontrado, a resposta contém a mensagem `"Nenhum registro localizado!"`.
- Se o autor **for** encontrado, seus dados são armazenados em `resposta.Dados` e a mensagem `"Autor Localizado!"` é retornada.
- Se ocorrer alguma exceção durante a busca, a mensagem de erro é armazenada e `Status` é definido como `false`.

### Fluxo do Código

1. Tenta buscar um autor no banco de dados pelo `idAutor`.
2. **Se encontrar** → retorna o autor e uma mensagem de sucesso.
3. **Se não encontrar** → retorna uma mensagem informando que não há registros.
4. **Se der erro** → captura a exceção e retorna a mensagem de erro.

Esse padrão — tentar a operação, tratar o caso de "não encontrado" separadamente do erro, e capturar exceções — é o que mantém o `ResponseModel<T>` sempre consistente, independentemente do resultado da busca.

## 2. Incluindo o Método no AutorController.cs

Com o `BuscarAutorPorId` implementado no Service, o próximo passo é expor esse método como um novo endpoint no `AutorController.cs`.

![BuscarAutorPorId no AutorController](https://github.com/joycequoos/Listar_Autor_Por_ID/blob/main/img/02_AutorController_BuscarId.png)

Seguindo o mesmo padrão do endpoint `ListarAutores`, esse novo método:

- É decorado com um atributo `[HttpGet(...)]`, definindo a rota do endpoint (por exemplo, recebendo o `id` como parâmetro na URL).
- Recebe o `id` do autor como parâmetro do método.
- Chama `_autorInterface.BuscarAutorPorId(id)`, delegando a lógica de busca ao Service, através da interface `IAutorInterface`.
- Retorna o resultado envolvido em `Ok(...)`, respeitando o padrão de resposta já utilizado nos demais endpoints (`ResponseModel<T>` com status HTTP `200`).

## 3. Executando o Projeto e Testando o Método

Com o método implementado no Controller, o projeto é executado novamente para testar o novo endpoint pelo Swagger — o mesmo processo já utilizado para testar o `ListarAutores`.

![Testando o Método BuscarAutorPorId](https://github.com/joycequoos/Listar_Autor_Por_ID/blob/main/img/03_Testar_Metodo_BuscarIdAutor.png)

No Swagger, o novo endpoint aparece listado junto ao grupo **Autor**. Ao expandi-lo e clicar em **Try it out**, é possível informar o ID do autor que se deseja buscar.

![Informando o ID e Executando a Busca](https://github.com/joycequoos/Listar_Autor_Por_ID/blob/main/img/04_BuscarId_2.png)

Após informar um ID válido e clicar em **Execute**, a requisição é enviada ao endpoint, que por sua vez aciona o Controller, o Service e, por fim, consulta o banco de dados.

![Resultado da Busca por ID](https://github.com/joycequoos/Listar_Autor_Por_ID/blob/main/img/04_Testar_Metodo_3.png)

Se o ID informado corresponder a um autor existente, a resposta retorna o status `200`, com os dados do autor em `dados` e a mensagem `"Autor Localizado!"` em `mensagem`. Caso o ID não exista no banco, a resposta ainda retorna `200`, porém com `dados` vazio e a mensagem `"Nenhum registro localizado!"` — já que, nesse caso, não se trata de um erro da aplicação, e sim de uma busca sem resultado.

## Resumo do Fluxo

1. **Service** — implementa a lógica de busca (`BuscarAutorPorId`), tratando os três cenários possíveis: sucesso, não encontrado e erro.
2. **Controller** — expõe a lógica do Service como um endpoint HTTP GET, recebendo o ID via parâmetro de rota.
3. **Teste** — via Swagger, informando um ID e validando se a resposta retorna o autor correto (ou a mensagem apropriada, caso não exista).
