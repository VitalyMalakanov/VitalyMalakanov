## 7. Примеры реализации MCP-сервера

В этом разделе представлены базовые примеры реализации MCP-сервера на языках Python и TypeScript, демонстрирующие основные концепции, такие как создание сервера, регистрация обработчиков запросов и подключение транспорта.

### 7.1 Пример на Python

Этот пример показывает простой MCP-сервер на Python, который использует стандартные потоки ввода/вывода (stdio) для связи и предоставляет один ресурс.

```python
import asyncio
import mcp.types as types  # Импорт определений типов MCP
from mcp.server import Server  # Импорт класса Server из SDK
from mcp.server.stdio import stdio_server  # Импорт функции для создания stdio транспорта

# Создание экземпляра сервера с именем "example-server"
app = Server("example-server")

# Регистрация обработчика для запроса 'list_resources'
# Этот метод будет вызван, когда клиент запросит список ресурсов.
@app.list_resources()
async def list_resources() -> list[types.Resource]:
    # Возвращаем список ресурсов
    return [
        types.Resource(
            uri="example://resource",  # URI ресурса
            name="Example Resource"     # Имя ресурса для отображения
        )
    ]

# Основная асинхронная функция для запуска сервера
async def main():
    # Используем stdio_server для получения потоков ввода/вывода
    async with stdio_server() as streams:
        # Запуск сервера с полученными потоками и опциями инициализации
        await app.run(
            streams[0],  # Поток для чтения (stdin)
            streams[1],  # Поток для записи (stdout)
            app.create_initialization_options()  # Создание стандартных опций инициализации
        )

# Точка входа в программу
if __name__ == "__main__":
    asyncio.run(main())  # Запуск асинхронного приложения
```
В данном примере:
1.  Инициализируется `Server` с уникальным именем.
2.  С помощью декоратора `@app.list_resources()` регистрируется асинхронная функция `list_resources`, которая будет обрабатывать запросы на получение списка ресурсов.
3.  Функция `main` настраивает `stdio_server` для коммуникации через стандартные потоки и запускает сервер `app.run()`, передавая ему эти потоки и опции инициализации.

### 7.2 Пример на TypeScript

Этот пример демонстрирует аналогичный MCP-сервер на TypeScript, также использующий stdio транспорт и предоставляющий один ресурс.

```typescript
import { Server } from "@modelcontextprotocol/sdk/server/index.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";
import { ListResourcesRequestSchema } from "@modelcontextprotocol/sdk/protocol"; // Предполагаемый импорт схемы запроса ListResources

// Создание экземпляра MCP сервера
// Указываем информацию о сервере (имя, версия) и его возможности.
const server = new Server({
  name: "example-server",       // Имя сервера
  version: "1.0.0"              // Версия сервера
}, {
  capabilities: {               // Определение возможностей сервера
    resources: {}               // Базовая возможность: поддержка ресурсов (детали могут быть расширены)
  }
});

// Регистрация обработчика для запросов типа ListResourcesRequestSchema.
// Этот обработчик будет вызван, когда клиент отправит запрос на получение списка ресурсов.
server.setRequestHandler(ListResourcesRequestSchema, async () => {
  // Возвращаем объект с полем 'resources', содержащим массив ресурсов.
  return {
    resources: [
      {
        uri: "example://resource",  // URI ресурса
        name: "Example Resource"     // Имя ресурса для отображения
      }
    ]
  };
});

// Настройка и подключение транспорта.
// В данном случае используется StdioServerTransport, который работает через стандартные потоки ввода/вывода.
const transport = new StdioServerTransport(); // Создание экземпляра Stdio транспорта
await server.connect(transport);               // Подключение сервера к транспорту для начала обмена сообщениями

console.log("MCP Server (TypeScript) started using Stdio transport."); // Сообщение о запуске сервера
```
В этом TypeScript примере:
1.  Создается экземпляр `Server`, передавая ему информацию о сервере и его возможностях.
2.  Метод `server.setRequestHandler()` используется для привязки обработчика к запросам, соответствующим `ListResourcesRequestSchema`. Обработчик асинхронно возвращает список ресурсов.
3.  Создается `StdioServerTransport`, и сервер подключается к этому транспорту с помощью `server.connect(transport)`.
4.  Добавлено сообщение в консоль для индикации того, что сервер запущен.

Эти примеры служат отправной точкой для создания более сложных MCP-серверов. Разработчики могут добавлять больше обработчиков для различных запросов и уведомлений, а также реализовывать более сложную логику внутри этих обработчиков.
