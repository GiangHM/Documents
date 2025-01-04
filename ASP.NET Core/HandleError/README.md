There are many way to handle error when working with ASP.Net Core: Developer exception page, Exception handler page, Exception handler lambda...
But Developer usually prefer a global approach for handling error.
Before ASP.Net 8.0, developer ofter use a CustomMiddleware for this purpose.

Starting from version 8.0 they can use a new way is: IExeptionFilter