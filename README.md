# Iniciando um projeto Node.js com TypeScript

- Abra o seu navegador de preferência, crie uma conta no https://github.com.
- Crie um repositório utilizando o botão verde na parte esquerda da tela. Deixe o repositório público, e deixe a caixa `Add a README file` (adicionar um arquivo README) habilitada.
- Agora, em seu computador, crie uma pasta para o projeto.
- Abra o VS Code (Visual Studio Code) e pressione o botão `Clone Git Repository` (Clonar Repositório Git). Se o seu VS Code não possuir esta opção, atualize-o.
- Vá até o seu navegador, copie (Ctrl+C) o endereço da página onde criou o repositório e cole (Ctrl+V) na caixa de texto na parte de cima da tela do VS Code.
- Escolha a pasta que você criou como a pasta a ser utilizada para o projeto. Após isso, clique em Open para abrí-la.
- Após ter aberto o projeto pelo VS Code, abre o terminal do VS Code, pressionando Ctrl e a tecla embaixo do esc (pode ser aspas ou outro botão, dependendo do teclado).
- Copie a caixa de texto abaixo utilizando o botão que parece dois quadrados, no canto superior direito da caixa.
- Cole o texto no terminal que apareceu na parte debaixo do VS Code e aperte enter. Isso irá executar os comandos, iniciando o npm, e instalando recursos necessários.

```bash
npm init -y
npm install express cors sqlite3 sqlite
npm install --save-dev typescript nodemon ts-node @types/express @types/cors
npx tsc --init
mkdir src
```

- Após isso, selecione a pasta src, clique com o botão direito do mouse e selecione `New File`, que deve ser a primeira opção.
- Nomeie este arquivo como `app.ts`

## Configuranado o `tsconfig.json`

- Abra o arquivo tsconfig.json, que os comandos npm devem ter criado.
- Selecione o texto do arquivo e pressione Ctrl+F. Isso irá abrir uma caixa de texto no canto superior direito da tela e você deve escrever `"outDir"`. Selecione a linha inteira clicando nela com o botão esquerdo do mouse três vezes e apague-a.
- Copie a caixa de texto seguinte e cole no lugar da linha apagada.

```json
   "outDir": "./dist",
   "rootDir": "./src",
```

- Você pode apagar as linhas comentadas (as que possuem // no começo) mas isso não é necessário. Se você não as apagou, ignore-as.
- Seu arquivo tsconfig.json deve estar assim (Ignore a diferença do `"target"`. Ele muda dependendo da versão de javascript instalada):

```json
{
  "compilerOptions": {
    "target": "ES2017",
    "module": "commonjs",
    "outDir": "./dist",
    "rootDir": "./src",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true
  }
}
```

## Configurando o `package.json`

- Embaixo da linha `"scripts"`, coloque uma vírgula no final da linha `"test"` e adicione esta linha:

```json
  "dev": "npx nodemon src/app.ts"
```

## Criando arquivo inicial do servidor

- Adicione o seguinte código ao arquivo `app.ts`:

```typescript
import express from 'express';
import cors from 'cors';

const port = 3333;
const app = express();

app.use(cors());
app.use(express.json());

app.get('/', (req, res) => {
  res.send('Hello World');
});

app.listen(port, () => {
  console.log(`Server running on port ${port}`);
});
```

## Inicializando o servidor

- Execute o comando seguinte no terminal do VS Code:

```bash
npm run dev
```

- Se tudo ocorrer bem, você verá a mensagem `Server running on port 3333` no terminal.

## Testando o servidor

- Abra o navegador e acesse `http://localhost:3333`, você deve ver a mensagem `Hello World` no canto superior esquerdo da tela.

## Configurando o banco de dados

- Crie um arquivo chamado `database.ts` dentro da pasta `src`.
- Copie o seguinte código e cole dentro do arquivo `database.ts`:

```typescript
import { open, Database } from 'sqlite';
import sqlite3 from 'sqlite3';

let instance: Database | null = null;

export async function connect() {
  if (instance) return instance;

  const db = await open({
     filename: './src/database.sqlite',
     driver: sqlite3.Database
   });
  
  await db.exec(`
    CREATE TABLE IF NOT EXISTS users (
      id INTEGER PRIMARY KEY AUTOINCREMENT,
      name TEXT,
      email TEXT
    )
  `);

  instance = db;
  return db;
}
```

## Adicionando o banco de dados ao servidor

- Apague tudo dentro do arquivo `app.ts`.
- Copie e cole o seguinte código ao `app.ts`:

```typescript
import express from 'express';
import cors from 'cors';
import { connect } from './database';

const port = 3333;
const app = express();

app.use(cors());
app.use(express.json());

app.get('/', (req, res) => {
  res.send('Hello World');
});

app.post('/users', async (req, res) => {
  const db = await connect();
  const { name, email } = req.body;

  const result = await db.run('INSERT INTO users (name, email) VALUES (?, ?)', [name, email]);
  const user = await db.get('SELECT * FROM users WHERE id = ?', [result.lastID]);

  res.json(user);
});

app.listen(port, () => {
  console.log(`Server running on port ${port}`);
});
```

## Inserindo dados

- Instale a extensão `REST Client` no VS Code.
- Crie uma pasta chamada `ts.http` dentro da pasta `src`.
- Copie e cole o seguinte código no arquivo `ts.http`:

```json
POST http://localhost:3333/users
Content-Type: application/json

{
  "name": "John Doe",
  "email": "lorem@ipsum.com"
}
```

- Após colar, aperte no botão ``Send Request`` que deve estar em cima do código.

## Listando os usuários

- Adicione a rota `/users` ao servidor. Para fazer isso, copie e cole o seguinte código ao final do `app.ts`:

```typescript
app.get('/users', async (req, res) => {
  const db = await connect();
  const users = await db.all('SELECT * FROM users');

  res.json(users);
});
```

## Testando a inserção de dados

- Caso o servidor já esteja aberto, feche-o apertando `Ctrl+C`, pressionando `S` ou `Y`, e  `Enter`.
- Para testar a inserção dos dados, execute o comando `npm run dev` para inicar o servidor.
- Quando o servidor iniciar, vá até a pasta `ts.http` e clique no "Send Request" em cima do texto do arquivo.
- Vá até o navegador e abra o website `http://localhost:3333/users`. O resultado deve ser:

```html
[{"id":1,"name":"John Doe","email":"lorem@ipsum.com"}]
```

- ou:

```html
id	1
name	"John Doe"
email	"lorem@ipsum.com"
```

## Editando um usuário

- Adicione a rota `/users/:id` ao servidor.
- Para isso, copie e cole o seguinte código no arquivo ``app.ts``:

```typescript
app.put('/users/:id', async (req, res) => {
  const db = await connect();
  const { name, email } = req.body;
  const { id } = req.params;

  await db.run('UPDATE users SET name = ?, email = ? WHERE id = ?', [name, email, id]);
  const user = await db.get('SELECT * FROM users WHERE id = ?', [id]);

  res.json(user);
});
```

## Deletando um usuário

- Adicione a rota `/users/:id` ao servidor.
- Para isso, copie e cole o seguinte código no arquivo ``app.ts``:

```typescript
app.delete('/users/:id', async (req, res) => {
  const db = await connect();
  const { id } = req.params;

  await db.run('DELETE FROM users WHERE id = ?', [id]);

  res.json({ message: 'User deleted' });
});
```

## Adicionando html estático

- Crie uma pasta chamada `public` no diretório principal (fora de `src`).
- Crie um arquivo dentro da pasta `public` e chame-a de `index.html`.
- Em `app.ts`, remova as linhas do primeiro `app.get`.
- Após isso, adicione `app.use(express.static(__dirname + '/../public'))` após os outros `app.use`.
- Isso irá enviar ao usuário todos os arquivos da pasta `public`.
- Copie e cole o seguinte código no arquivo `index.html`:

```HTML
<!DOCTYPE html>
<html lang="en">

<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Document</title>
</head>

<body>
  <form>
    <input type="text" name="name" placeholder="Nome">
    <input type="email" name="email" placeholder="Email">
    <button type="submit">Cadastrar</button>
  </form>

  <table>
    <thead>
      <tr>
        <th>Id</th>
        <th>Name</th>
        <th>Email</th>
        <th>Ações</th>
      </tr>
    </thead>
    <tbody>
      <!--  -->
    </tbody>
  </table>

  <script>
    // 
    const form = document.querySelector('form')

    form.addEventListener('submit', async (event) => {
      event.preventDefault()

      const name = form.name.value
      const email = form.email.value

      await fetch('/users', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ name, email })
      })

      form.reset()
      fetchData()
    })

    // 
    const tbody = document.querySelector('tbody')

    async function fetchData() {
      const resp = await fetch('/users')
      const data = await resp.json()

      tbody.innerHTML = ''

      data.forEach(user => {
        const tr = document.createElement('tr')
        tr.innerHTML = `
          <td>${user.id}</td>
          <td>${user.name}</td>
          <td>${user.email}</td>
          <td>
            <button class="excluir">excluir</button>
            <button class="editar">editar</button>
          </td>
        `

        const btExcluir = tr.querySelector('button.excluir')
        const btEditar = tr.querySelector('button.editar')

        btExcluir.addEventListener('click', async () => {
          await fetch(`/users/${user.id}`, { method: 'DELETE' })
          tr.remove()
        })

        btEditar.addEventListener('click', async () => {
          const name = prompt('Novo nome:', user.name)
          const email = prompt('Novo email:', user.email)

          await fetch(`/users/${user.id}`, {
            method: 'PUT',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify({ name, email })
          })

          fetchData()
        })

        tbody.appendChild(tr)
      })
    }

    fetchData()
  </script>
</body>

</html>
```

- Rode `npm run dev` e abra em um navegador o link `localhost:3333`.
- Agora, você pode adicionar, editar e remover contas de usuários.

