# 🏥 MedVoll – API da Clínica Médica (Segurança)
 
API REST em **.NET 9** para gerenciar **médicos** e **consultas** de uma clínica, construída com **ASP.NET Core**, **Entity Framework Core (SQLite)** e **ASP.NET Core Identity**.
 
Este repositório representa o estágio de **segurança** do projeto MedVoll: ele já traz a base de **usuários, perfis (roles) e claims** com Identity, além de autorização por perfil em endpoints sensíveis.
 
---
 
## ✨ Funcionalidades
 
- **Médicos**: listagem paginada, consulta por id, filtro por especialidade, cadastro, edição e exclusão.
- **Consultas**: listagem paginada (ordenada por data), cadastro, edição e exclusão, vinculadas a um médico.
- **Regras de negócio**: e-mail e CRM únicos por médico; consulta só pode ser registrada para um médico existente.
- **Validação de dados** com *Data Annotations* (nome, e-mail, CRM de 4 a 6 dígitos, telefone brasileiro e CPF de 11 dígitos para o paciente).
- **Identity**: criação automática de usuários, perfis `Admin` e `User` e claims na inicialização.
- **Autorização por perfil**: o cadastro, a edição e a exclusão de médicos exigem o perfil `Admin`.
- **Paginação** com 5 itens por página.
- Documentação interativa com **Swagger** (em ambiente de desenvolvimento).
## 🧰 Tecnologias
 
| Área | Tecnologias |
|---|---|
| Plataforma | C#, .NET 9, ASP.NET Core (Controllers) |
| Dados | Entity Framework Core 9, SQLite, Migrations |
| Segurança | ASP.NET Core Identity (usuários, roles e claims), pacote JWT Bearer |
| Documentação | Swashbuckle (Swagger) |
 
## 📁 Estrutura do projeto
 
```
MedVoll.sln
└── MedVoll.Web/
    ├── Controllers/     # MedicoController e ConsultaController (rotas api/[controller])
    ├── Services/        # Regras de negócio (MedicoService, ConsultaService)
    ├── Repositories/    # Acesso a dados (MedicoRepository, ConsultaRepository)
    ├── Interfaces/      # Contratos de serviços e repositórios
    ├── Models/          # Entidades: Medico, Consulta e enum Especialidade
    ├── Dtos/            # MedicoDto, ConsultaDto e PaginatedList
    ├── Data/            # ApplicationDbContext, IdentitySeeder e Migrations
    ├── Exceptions/      # RegraDeNegocioException
    ├── Extensions/      # Utilitários (enum e configuração do Swagger com Bearer)
    ├── appsettings.json # String de conexão e configurações de JWT
    └── vollmed.db       # Banco SQLite
```
 
## ✅ Pré-requisitos
 
- [.NET SDK 9.0](https://dotnet.microsoft.com/download/dotnet/9.0)
- (Opcional) Ferramenta do EF Core, para recriar o banco:
```bash
dotnet tool install --global dotnet-ef
```
 
## 🚀 Como executar
 
```bash
# 1. Clone o repositório
git clone https://github.com/<seu-usuario>/projeto-medvoll-seguranca-api-completo.git
cd projeto-medvoll-seguranca-api-completo
 
# 2. Execute a API
dotnet run --project MedVoll.Web --launch-profile https
```
 
- Swagger: `https://localhost:7058/Swagger/index.html`
- HTTP: `http://localhost:5241/Swagger/index.html`
O banco utiliza **SQLite** e fica em `MedVoll.Web/vollmed.db`, configurado em `ConnectionStrings:SqliteConnection`. O arquivo já acompanha o projeto, então a API roda sem configuração adicional.
 
### Recriando o banco (opcional)
 
Para gerar um banco novo com todas as migrations (médicos, consultas, especialidades e schema do Identity):
 
```bash
cd MedVoll.Web
dotnet ef database update
```
 
## 🔐 Usuários, perfis e claims
 
Na inicialização, o `IdentitySeeder` cria (se ainda não existirem) os perfis e usuários abaixo:
 
| Usuário | Senha | Perfis | Claims |
|---|---|---|---|
| `alice@smith.com` | `Password@123` | `User`, `Admin` | `FullName: Alice Smith`, `Role: Admin` |
| `bob@smith.com` | `Password@123` | `User` | `FullName: Bob Smith` |
 
## 📡 Endpoints
 
### Médicos – `api/Medico`
 
| Método | Rota | Descrição | Acesso |
|---|---|---|---|
| `GET` | `/api/Medico/Listar?page=1` | Lista médicos de forma paginada | Livre |
| `GET` | `/api/Medico/formulario/{id?}` | Dados de um médico (ou formulário vazio) | Livre |
| `GET` | `/api/Medico/especialidade/{especialidade}` | Lista médicos por especialidade | Livre |
| `POST` | `/api/Medico` | Cadastra, edita ou exclui (`_method=delete`) médico, via `form-data` | `Admin` |
 
### Consultas – `api/Consulta`
 
| Método | Rota | Descrição | Acesso |
|---|---|---|---|
| `GET` | `/api/Consulta/listar?page=1` | Lista consultas de forma paginada | Livre |
| `GET` | `/api/Consulta/formulario/{id?}` | Dados da consulta e lista de médicos | Livre |
| `POST` | `/api/Consulta/Salvar` | Cadastra, edita ou exclui (`_method=delete`) consulta, via `form-data` | Livre |
 
**Especialidades válidas:** `Cardiologia`, `Neurocirurgia`, `CirurgiaGeral`, `Pediatria`, `Oncologia` e `Diagnostico`.
 
### Regras de validação
 
| Campo | Regra |
|---|---|
| Médico – Nome | Obrigatório, mínimo de 5 caracteres |
| Médico – E-mail | Obrigatório, formato válido e único |
| Médico – CRM | Obrigatório, de 4 a 6 dígitos e único |
| Médico – Telefone | Obrigatório, formatos como `11999999999` ou `(11) 99999-9999` |
| Consulta – Paciente | Obrigatório, CPF com 11 dígitos |
| Consulta – Médico e Data | Obrigatórios |
 
## 🗺️ Status e próximos passos
 
Este projeto está em evolução, e a parte de segurança ainda está sendo concluída:
 
- [ ] Expor um endpoint de **login** que gere o token **JWT** (o pacote `JwtBearer` e as chaves `JWTKey` / `JWTTokenConfiguration` já estão no projeto, mas a autenticação JWT ainda não está registrada no `Program.cs`).
- [ ] Habilitar a configuração do **Swagger com Bearer** (`ConfigureSwaggerExtensions.cs`, hoje excluída da compilação).
- [ ] Proteger também os endpoints de **Consulta** com `[Authorize]` e políticas por perfil ou claim.
- [ ] Rever o `[ValidateAntiForgeryToken]` do `POST /api/Medico`, que não é típico de APIs com token.
- [ ] Retornar `404`/`400` mais adequados em erros de regra de negócio e validação.
- [ ] Adicionar testes automatizados.
## ⚠️ Segurança
 
Os valores presentes no `appsettings.json` (como a chave do JWT) e as senhas dos usuários do seeder são **exemplos para estudo**. Em qualquer ambiente real:
 
- não versione segredos: use **User Secrets**, variáveis de ambiente ou um cofre de segredos;
- gere uma chave JWT forte e exclusiva;
- remova ou altere os usuários criados pelo `IdentitySeeder`;
- não distribua o arquivo `vollmed.db` com dados reais.
## 📄 Licença
 
Defina aqui a licença do projeto (por exemplo, [MIT](https://opensource.org/licenses/MIT)).
 
---
 
Feito com 💙 durante os estudos de **.NET, Identity e segurança de APIs**.
 
