# Interface, Service, Controller and Endpoint: Find Author by ID

[← Back](https://github.com/joycequoos/WEB-API-com-.NET-8-e-SQL-Server)

<img width="276" height="462" alt="image" src="https://github.com/user-attachments/assets/8b090407-6256-4fdf-a047-e695eb56b6f7" />


Continuing on from the authors listing endpoint, this step implements the search for a specific author by their **ID**, following the same flow already established: Service → Controller → Endpoint test.

## 1. Implementing the Method in AutorService.cs

The `BuscarAutorPorId` method, created as an interface in the previous repository, is implemented in `AutorService.cs`. It is an asynchronous method that searches the database for an author based on the given ID (`idAutor`).

![BuscarAutorPorId Method in AutorService](https://github.com/joycequoos/Listar_Autor_Por_ID/blob/main/img/AutorService_BuscarID.png)

### Code Explanation

- The method returns an object of type `ResponseModel<AutorModel>`.
- It uses Entity Framework Core to access `_context.Autores`, a `DbSet<AutorModel>` that represents the authors table in the database.
- The search is performed with `FirstOrDefaultAsync`, which returns the first author with the provided ID, or `null` if none is found.
- If the author **is not** found, the response contains the message `"Nenhum registro localizado!"` ("No record found!").
- If the author **is** found, its data is stored in `resposta.Dados` and the message `"Autor Localizado!"` ("Author found!") is returned.
- If an exception occurs during the search, the error message is stored and `Status` is set to `false`.

### Code Flow

1. Attempts to find an author in the database by `idAutor`.
2. **If found** → returns the author and a success message.
3. **If not found** → returns a message stating that there are no records.
4. **If an error occurs** → catches the exception and returns the error message.

This pattern — attempting the operation, handling the "not found" case separately from the error, and catching exceptions — is what keeps the `ResponseModel<T>` consistent at all times, regardless of the search result.

## 2. Adding the Method to AutorController.cs

With `BuscarAutorPorId` implemented in the Service, the next step is to expose this method as a new endpoint in `AutorController.cs`.

![BuscarAutorPorId in AutorController](https://github.com/joycequoos/Listar_Autor_Por_ID/blob/main/img/02_AutorController_BuscarId.png)

Following the same pattern as the `ListarAutores` endpoint, this new method:

- Is decorated with an `[HttpGet(...)]` attribute, defining the endpoint's route (for example, receiving the `id` as a parameter in the URL).
- Receives the author's `id` as a method parameter.
- Calls `_autorInterface.BuscarAutorPorId(id)`, delegating the search logic to the Service, through the `IAutorInterface` interface.
- Returns the result wrapped in `Ok(...)`, following the same response pattern already used in the other endpoints (`ResponseModel<T>` with HTTP status `200`).

## 3. Running the Project and Testing the Method

With the method implemented in the Controller, the project is run again to test the new endpoint through Swagger — the same process already used to test `ListarAutores`.

![Testing the BuscarAutorPorId Method](https://github.com/joycequoos/Listar_Autor_Por_ID/blob/main/img/03_Testar_Metodo_BuscarIdAutor.png)

In Swagger, the new endpoint appears listed under the **Autor** group. By expanding it and clicking **Try it out**, you can enter the ID of the author you want to search for.

![Entering the ID and Running the Search](https://github.com/joycequoos/Listar_Autor_Por_ID/blob/main/img/04_BuscarId_2.png)

After entering a valid ID and clicking **Execute**, the request is sent to the endpoint, which in turn triggers the Controller, the Service, and finally queries the database.

![Search by ID Result](https://github.com/joycequoos/Listar_Autor_Por_ID/blob/main/img/04_Testar_Metodo_3.png)

If the provided ID matches an existing author, the response returns status `200`, with the author's data in `dados` and the message `"Autor Localizado!"` in `mensagem`. If the ID does not exist in the database, the response still returns `200`, but with an empty `dados` and the message `"Nenhum registro localizado!"` — since, in this case, it is not an application error, but rather a search with no result.

## Flow Summary

1. **Service** — implements the search logic (`BuscarAutorPorId`), handling the three possible scenarios: success, not found, and error.
2. **Controller** — exposes the Service logic as an HTTP GET endpoint, receiving the ID via a route parameter.
3. **Test** — via Swagger, entering an ID and validating whether the response returns the correct author (or the appropriate message, if it doesn't exist).
