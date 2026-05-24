# Guida Completa — AutoveloxProject
> Guida passo-passo per la creazione della soluzione AutoveloxProject su SQL Server e .NET 10.
> **Prerequisiti:** SQL Server 2025 Developer, SSMS 22, Visual Studio 2026 Community installati.

---

## INDICE

1. [Ripristino del Database Autovelox](#1-ripristino-del-database-autovelox)
2. [Creazione del Diagramma e delle Relazioni](#2-creazione-del-diagramma-e-delle-relazioni)
3. [Creazione del SQL Login swd2426](#3-creazione-del-sql-login-swd2426)
4. [Verifica e Query SQL](#4-verifica-e-query-sql)
5. [Creazione della Soluzione Visual Studio](#5-creazione-della-soluzione-visual-studio)
6. [Progetto Autovelox.Data — Class Library](#6-progetto-autoveloxdata--class-library)
7. [Scaffold Database-First](#7-scaffold-database-first)
8. [Progetto Autovelox.WebAPI](#8-progetto-autoveloxwebapi)
9. [Configurazione Program.cs e appsettings.json](#9-configurazione-programcs-e-appsettingsjson)
10. [Creazione dei Controller](#10-creazione-dei-controller)
11. [Test con Swagger](#11-test-con-swagger)

---

## 1. Ripristino del Database Autovelox

### 1.1 Spostare il file di backup

Prima di tutto sposta il file `Autovelox.bak` nella cartella di backup di SQL Server.

1. Apri **Esplora File** di Windows
2. Naviga in:
   ```
   C:\Program Files\Microsoft SQL Server\MSSQL17.MSSQLSERVER\MSSQL\Backup
   ```
3. Windows potrebbe chiedere autorizzazione — clicca **Continua**
4. Copia o sposta il file `Autovelox.bak` in questa cartella

### 1.2 Connettersi a SQL Server con SSMS

1. Apri **SQL Server Management Studio 22** dal menu Start
2. Alla schermata di login, clicca **"Ignorare e aggiungere account in un secondo momento"**
3. Si apre la finestra **Connetti** — compila così:
   - **Nome server:** `localhost`
   - **Autenticazione:** `Autenticazione di Windows`
   - **Crittografa:** `Facoltativo`
4. Clicca **Connetti**

Dovresti vedere nell'Esplora oggetti: `localhost (SQL Server 17.0.1000.7 - NOMETUO\nomeutente)`

### 1.3 Abilitare l'autenticazione mista (SQL Server + Windows)

Per poter usare il login SQL `swd2426` in seguito, dobbiamo abilitare la modalità mista:

1. Nell'Esplora oggetti, fai **click destro** sul nodo `localhost` in cima
2. Seleziona **Proprietà**
3. Clicca su **Sicurezza** nel pannello a sinistra
4. Seleziona **"Autenticazione di SQL Server e di Windows"**
5. Clicca **OK**
6. Apri **Gestione configurazione SQL Server** (cerca `SQLServerManager17.msc` nel menu Start)
7. Clicca su **Servizi di SQL Server**
8. Fai **click destro** su **SQL Server (MSSQLSERVER)** → **Riavvia**

### 1.4 Ripristinare il database

1. In SSMS, nell'Esplora oggetti fai **click destro** su **Database**
2. Seleziona **"Ripristina database..."**
3. Nella finestra che si apre:
   - Seleziona **Dispositivo** come origine
   - Clicca sui **...** e aggiungi il file `Autovelox.bak`
   - SSMS rileverà automaticamente il nome database: **Autovelox**
4. Clicca **OK**
5. Apparirà il messaggio: *"Il database è stato ripristinato correttamente"*

Per verificare: espandi **Database** nell'Esplora oggetti e vedrai **Autovelox** con le tabelle:
- `dbo.Comune`
- `dbo.Mappa`
- `dbo.Provincia`
- `dbo.Regione`
- `dbo.RipartizioneGeografica`

---

## 2. Creazione del Diagramma e delle Relazioni

### 2.1 Correggere il proprietario del database

Prima di creare il diagramma, esegui questa query (click su **Nuova query**, seleziona database **Autovelox** dal dropdown):

```sql
USE Autovelox
GO
EXEC sp_changedbowner 'ROTOCALCO\aless'  -- sostituisci con il tuo NomePC\nomeutente
GO
```

### 2.2 Creare il diagramma

1. Nell'Esplora oggetti, espandi **Autovelox** → fai **click destro** su **Diagrammi database**
2. Seleziona **"Nuovo diagramma database"**
3. Aggiungi tutte le tabelle al diagramma
4. Le relazioni già esistenti saranno visibili:
   - `Regione` → `RipartizioneGeografica`
   - `Provincia` → `Regione`
   - `Comune` → `Provincia`

### 2.3 Aggiungere la relazione mancante (Mappa → Comune)

La relazione tra `Mappa` e `Comune` non esiste ancora — dobbiamo crearla:

1. Nel diagramma, clicca sulla colonna **`IdComune`** nella tabella **Mappa**
2. Tieni premuto il mouse e **trascina** verso la colonna **`IdComune`** nella tabella **Comune**
3. Si apre la finestra **Tabelle e colonne** — verifica:
   - **Nome relazione:** `FK_Mappa_Comune`
   - **Tabella di chiave primaria:** `Comune` → `IdComune`
   - **Tabella di chiave esterna:** `Mappa` → `IdComune`
4. Clicca **OK** → **OK**
5. Salva il diagramma con **CTRL+S**, nome: `DiagrammaAutovelox`

Le relazioni finali devono essere:

| Tabella figlia | Tabella padre | Chiave esterna |
|---|---|---|
| Mappa | Comune | IdComune |
| Comune | Provincia | IdProvincia |
| Provincia | Regione | IdRegione |
| Regione | RipartizioneGeografica | IdRipartizioneGeografica |

---

## 3. Creazione del SQL Login swd2426

### 3.1 Creare il login

1. Nell'Esplora oggetti, espandi **Sicurezza** (a livello server, non dentro Autovelox)
2. Espandi **Account di accesso**
3. Fai **click destro** su **Account di accesso** → **Nuovo account di accesso...**
4. Compila la scheda **Generale**:
   - **Nome account di accesso:** `swd2426`
   - **Autenticazione:** seleziona **Autenticazione di SQL**
   - **Password:** `its-2026`
   - **Conferma password:** `its-2026`
   - Deseleziona **"Applica criteri password"**
   - **Database predefinito:** `master`

### 3.2 Mappare l'utente su Autovelox

1. Clicca su **Mapping utente** nel pannello a sinistra
2. Spunta la casella accanto ad **Autovelox**
3. In basso nella sezione **"Appartenenza a ruoli del database per: Autovelox"**, spunta **`db_owner`**
4. Clicca **OK**

### 3.3 Verificare i permessi

Apri una nuova query e verifica che `swd2426` sia mappato solo su Autovelox:

```sql
SELECT 
    dp.name AS User_Name,
    rp.name AS Role_Name
FROM Autovelox.sys.database_principals dp
JOIN Autovelox.sys.database_role_members rm ON dp.principal_id = rm.member_principal_id
JOIN Autovelox.sys.database_principals rp ON rm.role_principal_id = rp.principal_id
WHERE dp.name = 'swd2426'
```

Risultato atteso: `swd2426 | db_owner`

---

## 4. Verifica e Query SQL

Tutte le query seguenti vanno eseguite **connessi con l'account `swd2426`**.

Per connettersi con `swd2426`:
1. Nell'Esplora oggetti clicca **Connetti** → **Motore di database**
2. **Nome server:** `localhost`
3. **Autenticazione:** `Autenticazione di SQL Server`
4. **Account di accesso:** `swd2426`
5. **Password:** `its-2026`
6. **Crittografa:** `Facoltativo`

### 4.1 Correzione Latitudine e Longitudine

I valori sono memorizzati come interi (es. `75781483` invece di `7.5781483`). Esegui:

```sql
USE Autovelox;
GO

UPDATE Mappa SET Latitudine = Latitudine / 10.0 WHERE Latitudine > 47;
UPDATE Mappa SET Latitudine = Latitudine / 10.0 WHERE Latitudine > 47;
UPDATE Mappa SET Latitudine = Latitudine / 10.0 WHERE Latitudine > 47;
UPDATE Mappa SET Latitudine = Latitudine / 10.0 WHERE Latitudine > 47;
UPDATE Mappa SET Latitudine = Latitudine / 10.0 WHERE Latitudine > 47;
UPDATE Mappa SET Latitudine = Latitudine / 10.0 WHERE Latitudine > 47;
UPDATE Mappa SET Latitudine = Latitudine / 10.0 WHERE Latitudine > 47;
UPDATE Mappa SET Latitudine = Latitudine / 10.0 WHERE Latitudine > 47;

UPDATE Mappa SET Longitudine = Longitudine / 10.0 WHERE Longitudine > 18;
UPDATE Mappa SET Longitudine = Longitudine / 10.0 WHERE Longitudine > 18;
UPDATE Mappa SET Longitudine = Longitudine / 10.0 WHERE Longitudine > 18;
UPDATE Mappa SET Longitudine = Longitudine / 10.0 WHERE Longitudine > 18;
UPDATE Mappa SET Longitudine = Longitudine / 10.0 WHERE Longitudine > 18;
UPDATE Mappa SET Longitudine = Longitudine / 10.0 WHERE Longitudine > 18;
UPDATE Mappa SET Longitudine = Longitudine / 10.0 WHERE Longitudine > 18;
UPDATE Mappa SET Longitudine = Longitudine / 10.0 WHERE Longitudine > 18;
```

Verifica il risultato (valori attesi: Lat 37.49–46.63, Long 7.51–17.20):

```sql
SELECT MIN(Latitudine), MAX(Latitudine), MIN(Longitudine), MAX(Longitudine)
FROM Mappa;
```

### 4.2 Query 1 — 100 record Piemonte

```sql
SELECT m.*
FROM Mappa m
INNER JOIN Comune c ON m.IdComune = c.IdComune
INNER JOIN Provincia p ON c.IdProvincia = p.Id
INNER JOIN Regione r ON p.IdRegione = r.Id
WHERE r.Denominazione = 'Piemonte';
-- Risultato atteso: 100 record
```

### 4.3 Query 2 — Autovelox sul 45° parallelo

```sql
SELECT *
FROM Mappa
WHERE Latitudine >= 45.0 AND Latitudine < 46.0;
```

### 4.4 Query 3 — Comuni con autovelox nella provincia di Teramo

```sql
SELECT c.Denominazione AS Comune
FROM Mappa m
INNER JOIN Comune c ON m.IdComune = c.IdComune
INNER JOIN Provincia p ON c.IdProvincia = p.Id
WHERE p.Denominazione = 'Teramo'
GROUP BY c.Denominazione
HAVING COUNT(m.Id) >= 1
ORDER BY c.Denominazione;
-- Risultato atteso: Francavilla al Mare
```

### 4.5 Query 4 — Anno installazione e numero comuni (GROUP BY)

```sql
SELECT 
    m.AnnoInserimento AS Anno,
    COUNT(DISTINCT m.IdComune) AS NumeroComuni
FROM Mappa m
GROUP BY m.AnnoInserimento
ORDER BY m.AnnoInserimento;
```

---

## 5. Creazione della Soluzione Visual Studio

1. Apri **Visual Studio 2026**
2. Dalla schermata iniziale clicca **"Crea un nuovo progetto"**
3. Cerca `Libreria di classi` nella barra di ricerca
4. Seleziona **"Libreria di classi"** con tag C#, Windows, Libreria (NON quella per Azure U-SQL)
5. Clicca **Avanti**
6. Compila:
   - **Nome del progetto:** `Autovelox.Data`
   - **Percorso:** `C:\Users\tuonome\source\repos` (o dove preferisci)
   - **Nome soluzione:** `AutoveloxProject`
   - **NON** spuntare "Inserisci soluzione e progetto nella stessa directory"
7. Clicca **Avanti**
8. **Framework:** `.NET 10.0 (Supporto a lungo termine)`
9. Clicca **Crea**

Una volta creato il progetto, **elimina** il file `Class1.cs` (click destro → Elimina).

---

## 6. Progetto Autovelox.Data — Class Library

### 6.1 Installare i pacchetti NuGet

1. Nell'Esplora soluzioni, fai **click destro** su `Autovelox.Data`
2. Seleziona **"Gestisci pacchetti NuGet..."**
3. Vai alla scheda **Sfoglia**
4. Installa i seguenti pacchetti (cerca, seleziona, clicca Installa, poi Applica):
   - `Microsoft.EntityFrameworkCore.SqlServer` (versione 10.x.x)
   - `Microsoft.EntityFrameworkCore.Tools` (versione 10.x.x)
   - `Microsoft.EntityFrameworkCore.Design` (versione 10.x.x)
   - `Microsoft.EntityFrameworkCore` (versione 10.x.x)

### 6.2 Installare dotnet-ef globalmente

Apri **Strumenti → Gestione pacchetti NuGet → Console di Gestione pacchetti** ed esegui:

```
dotnet tool install --global dotnet-ef
```

---

## 7. Scaffold Database-First

Lo Scaffold genera automaticamente le classi C# (modelli) e il DbContext a partire dal database esistente.

Nella **Console di Gestione Pacchetti** (assicurati che il progetto predefinito sia `Autovelox.Data`), esegui:

```
Scaffold-DbContext "Server=localhost;Database=Autovelox;User Id=swd2426;Password=its-2026;TrustServerCertificate=True;" Microsoft.EntityFrameworkCore.SqlServer -OutputDir Models
```

Questo comando:
- Si connette ad Autovelox con l'account `swd2426`
- Genera una classe C# per ogni tabella nella cartella `Models`
- Genera `AutoveloxContext.cs` — il DbContext che gestisce la connessione

Dopo lo Scaffold, la cartella `Models` conterrà:
- `AutoveloxContext.cs`
- `Comune.cs`
- `Mappa.cs`
- `Provincium.cs` *(EF ha pluralizzato "Provincia" — è normale)*
- `Regione.cs`
- `RipartizioneGeografica.cs`

### 7.1 Rimuovere la connection string hardcoded

Apri `AutoveloxContext.cs` e sostituisci il metodo `OnConfiguring` con:

```csharp
protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
{
    if (!optionsBuilder.IsConfigured)
    {
        // Connection string configurata in appsettings.json tramite Program.cs
    }
}
```

---

## 8. Progetto Autovelox.WebAPI

### 8.1 Creare il progetto

1. Nell'Esplora soluzioni, fai **click destro** sulla **Soluzione `AutoveloxProject`** (non sul progetto)
2. Seleziona **Aggiungi → Nuovo progetto**
3. Cerca `ASP.NET Core Web API`
4. Seleziona **"API Web ASP.NET Core"** con tag C#, Windows, Web API
5. Clicca **Avanti**
6. **Nome del progetto:** `Autovelox.WebAPI`
7. Clicca **Avanti**
8. Impostazioni:
   - **Framework:** `.NET 10.0`
   - **Tipo di autenticazione:** Nessuna
   - **Configura per HTTPS:** ✅
   - **Abilita supporto OpenAPI:** ✅
   - **Usa i controller:** ✅
9. Clicca **Crea**

Dopo la creazione, **elimina** i file:
- `WeatherForecast.cs`
- `Controllers/WeatherForecastController.cs`

### 8.2 Aggiungere riferimento a Autovelox.Data

1. Nell'Esplora soluzioni, fai **click destro** su **Dipendenze** dentro `Autovelox.WebAPI`
2. Seleziona **"Aggiungi riferimento progetto..."**
3. Spunta **Autovelox.Data**
4. Clicca **OK**

### 8.3 Installare Swagger

Nella Console di Gestione Pacchetti, assicurati che il **Progetto predefinito** sia `Autovelox.WebAPI`, poi esegui:

```
Install-Package Swashbuckle.AspNetCore
```

### 8.4 Impostare Autovelox.WebAPI come progetto di avvio

Nell'Esplora soluzioni, fai **click destro** su `Autovelox.WebAPI` → **"Imposta come progetto di avvio"**

---

## 9. Configurazione Program.cs e appsettings.json

### 9.1 appsettings.json

Apri `appsettings.json` dentro `Autovelox.WebAPI` e sostituisci tutto il contenuto con:

```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=localhost;Database=Autovelox;User ID=swd2426;Password=its-2026;TrustServerCertificate=True"
  },
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning"
    }
  },
  "AllowedHosts": "*"
}
```

### 9.2 Program.cs

Apri `Program.cs` dentro `Autovelox.WebAPI` e sostituisci tutto il contenuto con:

```csharp
using Autovelox.Data.Models;
using Microsoft.EntityFrameworkCore;

namespace Autovelox.WebAPI
{
    public class Program
    {
        public static void Main(string[] args)
        {
            var builder = WebApplication.CreateBuilder(args);

            // Collega il DbContext al database SQL Server usando la stringa in appsettings.json
            builder.Services.AddDbContext<AutoveloxContext>(options =>
                options.UseSqlServer(builder.Configuration.GetConnectionString("DefaultConnection")));

            // Add services to the container.
            builder.Services.AddControllers()
                .AddJsonOptions(options =>
                {
                    // Evita errori di cicli circolari nella serializzazione JSON
                    options.JsonSerializerOptions.ReferenceHandler =
                        System.Text.Json.Serialization.ReferenceHandler.IgnoreCycles;
                });

            // Abilita l'esplorazione degli endpoint per Swagger
            builder.Services.AddEndpointsApiExplorer();

            // Genera il documento Swagger
            builder.Services.AddSwaggerGen();

            var app = builder.Build();

            // Configure the HTTP request pipeline.
            if (app.Environment.IsDevelopment())
            {
                app.UseSwagger();
                app.UseSwaggerUI();
            }

            app.UseHttpsRedirection();
            app.UseAuthorization();
            app.MapControllers();
            app.Run();
        }
    }
}
```

---

## 10. Creazione dei Controller

Per ogni controller: fai **click destro** sulla cartella `Controllers` dentro `Autovelox.WebAPI` → **Aggiungi → Controller...** → **"Controller API - Vuoto"** → dai il nome indicato → clicca **Aggiungi**.

### 10.1 RipartizioniGeograficheController.cs

```csharp
using Autovelox.Data.Models;
using Microsoft.AspNetCore.Mvc;
using Microsoft.EntityFrameworkCore;

namespace Autovelox.WebAPI.Controllers
{
    [Route("api/[controller]")]
    [ApiController]
    public class RipartizioniGeograficheController : ControllerBase
    {
        private readonly AutoveloxContext _context;

        public RipartizioniGeograficheController(AutoveloxContext context)
        {
            _context = context;
        }

        // GET: api/RipartizioniGeografiche
        [HttpGet]
        public async Task<ActionResult<IEnumerable<RipartizioneGeografica>>> Get()
        {
            return await _context.RipartizioneGeograficas.ToListAsync();
        }

        // GET api/RipartizioniGeografiche/5
        [HttpGet("{id}")]
        public async Task<ActionResult<RipartizioneGeografica>> Get(int id)
        {
            var ripartizione = await _context.RipartizioneGeograficas.FindAsync(id);

            if (ripartizione == null)
                return NotFound();

            return ripartizione;
        }
    }
}
```

### 10.2 RegioniController.cs

```csharp
using Autovelox.Data.Models;
using Microsoft.AspNetCore.Mvc;
using Microsoft.EntityFrameworkCore;

namespace Autovelox.WebAPI.Controllers
{
    [Route("api/[controller]")]
    [ApiController]
    public class RegioniController : ControllerBase
    {
        private readonly AutoveloxContext _context;

        public RegioniController(AutoveloxContext context)
        {
            _context = context;
        }

        // GET: api/Regioni
        [HttpGet]
        public async Task<ActionResult<IEnumerable<Regione>>> Get()
        {
            return await _context.Regiones.ToListAsync();
        }

        // GET api/Regioni/5
        [HttpGet("{id}")]
        public async Task<ActionResult<Regione>> Get(int id)
        {
            var regione = await _context.Regiones.FindAsync(id);

            if (regione == null)
                return NotFound();

            return regione;
        }

        // GET api/Regioni/5/Province
        [HttpGet("{id}/Province")]
        public async Task<ActionResult<IEnumerable<Provincium>>> GetProvince(int id)
        {
            var province = await _context.Provincia
                .Where(p => p.IdRegione == id)
                .ToListAsync();

            return province;
        }
    }
}
```

### 10.3 ProvinceController.cs

```csharp
using Autovelox.Data.Models;
using Microsoft.AspNetCore.Mvc;
using Microsoft.EntityFrameworkCore;

namespace Autovelox.WebAPI.Controllers
{
    [Route("api/[controller]")]
    [ApiController]
    public class ProvinceController : ControllerBase
    {
        private readonly AutoveloxContext _context;

        public ProvinceController(AutoveloxContext context)
        {
            _context = context;
        }

        // GET: api/Province
        [HttpGet]
        public async Task<ActionResult<IEnumerable<Provincium>>> Get()
        {
            return await _context.Provincia.ToListAsync();
        }

        // GET api/Province/5
        [HttpGet("{id}")]
        public async Task<ActionResult<Provincium>> Get(int id)
        {
            var provincia = await _context.Provincia.FindAsync(id);

            if (provincia == null)
                return NotFound();

            return provincia;
        }

        // GET api/Province/5/Comuni
        [HttpGet("{id}/Comuni")]
        public async Task<ActionResult<IEnumerable<Comune>>> GetComuni(int id)
        {
            var comuni = await _context.Comunes
                .Where(c => c.IdProvincia == id)
                .ToListAsync();

            return comuni;
        }
    }
}
```

### 10.4 ComuniController.cs

```csharp
using Autovelox.Data.Models;
using Microsoft.AspNetCore.Mvc;
using Microsoft.EntityFrameworkCore;

namespace Autovelox.WebAPI.Controllers
{
    [Route("api/[controller]")]
    [ApiController]
    public class ComuniController : ControllerBase
    {
        private readonly AutoveloxContext _context;

        public ComuniController(AutoveloxContext context)
        {
            _context = context;
        }

        // GET: api/Comuni
        [HttpGet]
        public async Task<ActionResult<IEnumerable<Comune>>> Get()
        {
            return await _context.Comunes.ToListAsync();
        }

        // GET api/Comuni/5
        [HttpGet("{id}")]
        public async Task<ActionResult<Comune>> Get(int id)
        {
            var comune = await _context.Comunes.FindAsync(id);

            if (comune == null)
                return NotFound();

            return comune;
        }

        // GET api/Comuni/5/Mappe
        [HttpGet("{id}/Mappe")]
        public async Task<ActionResult<IEnumerable<Mappa>>> GetMappe(int id)
        {
            var mappe = await _context.Mappas
                .Where(m => m.IdComune == id)
                .ToListAsync();

            return mappe;
        }
    }
}
```

### 10.5 MappeController.cs

```csharp
using Autovelox.Data.Models;
using Microsoft.AspNetCore.Mvc;
using Microsoft.EntityFrameworkCore;

namespace Autovelox.WebAPI.Controllers
{
    [Route("api/[controller]")]
    [ApiController]
    public class MappeController : ControllerBase
    {
        private readonly AutoveloxContext _context;

        public MappeController(AutoveloxContext context)
        {
            _context = context;
        }

        // GET: api/Mappe
        // Elenco di tutti gli autovelox d'Italia
        [HttpGet]
        public async Task<ActionResult<IEnumerable<Mappa>>> Get()
        {
            return await _context.Mappas
                .Include(m => m.IdComuneNavigation)
                .ToListAsync();
        }

        // GET api/Mappe/5
        // Dettaglio singola postazione con tutte le informazioni joined
        [HttpGet("{id}")]
        public async Task<ActionResult<Mappa>> Get(int id)
        {
            var mappa = await _context.Mappas
                .Include(m => m.IdComuneNavigation)
                    .ThenInclude(c => c.IdProvinciaNavigation)
                        .ThenInclude(p => p.IdRegioneNavigation)
                            .ThenInclude(r => r.IdRipartizioneGeograficaNavigation)
                .FirstOrDefaultAsync(m => m.Id == id);

            if (mappa == null)
                return NotFound();

            return mappa;
        }

        // GET api/Mappe/cerca?regione=Piemonte&provincia=Torino&comune=Collegno
        // Ricerca per regione, provincia, comune (tutti opzionali)
        [HttpGet("cerca")]
        public async Task<ActionResult<IEnumerable<Mappa>>> Cerca(
            [FromQuery] string? regione,
            [FromQuery] string? provincia,
            [FromQuery] string? comune)
        {
            var query = _context.Mappas
                .Include(m => m.IdComuneNavigation)
                    .ThenInclude(c => c.IdProvinciaNavigation)
                        .ThenInclude(p => p.IdRegioneNavigation)
                .AsQueryable();

            if (!string.IsNullOrEmpty(comune))
                query = query.Where(m => m.IdComuneNavigation.Denominazione == comune);

            if (!string.IsNullOrEmpty(provincia))
                query = query.Where(m => m.IdComuneNavigation.IdProvinciaNavigation.Denominazione == provincia);

            if (!string.IsNullOrEmpty(regione))
                query = query.Where(m => m.IdComuneNavigation.IdProvinciaNavigation.IdRegioneNavigation.Denominazione == regione);

            return await query.ToListAsync();
        }
    }
}
```

---

## 11. Test con Swagger

### 11.1 Avviare l'applicazione

1. Clicca il pulsante **▶ https** verde nella toolbar in alto
2. Si apre una finestra terminale con i log — cerca la riga:
   ```
   Now listening on: https://localhost:7256
   ```

### 11.2 Aprire Swagger UI

Apri il browser e vai a:
```
https://localhost:7256/swagger
```

Dovresti vedere l'interfaccia Swagger con tutti gli endpoint:

| Controller | Endpoint |
|---|---|
| Comuni | GET /api/Comuni |
| Comuni | GET /api/Comuni/{id} |
| Comuni | GET /api/Comuni/{id}/Mappe |
| Mappe | GET /api/Mappe |
| Mappe | GET /api/Mappe/{id} |
| Mappe | GET /api/Mappe/cerca |
| Province | GET /api/Province |
| Province | GET /api/Province/{id} |
| Province | GET /api/Province/{id}/Comuni |
| Regioni | GET /api/Regioni |
| Regioni | GET /api/Regioni/{id} |
| Regioni | GET /api/Regioni/{id}/Province |
| RipartizioniGeografiche | GET /api/RipartizioniGeografiche |
| RipartizioniGeografiche | GET /api/RipartizioniGeografiche/{id} |

### 11.3 Testare un endpoint

1. Clicca su **GET /api/Mappe/cerca**
2. Clicca **"Try it out"**
3. Nel campo `regione` scrivi `Piemonte`
4. Clicca **Execute**
5. Dovresti ricevere una risposta **200 OK** con i 100 autovelox del Piemonte in formato JSON

---

## Struttura finale della soluzione

```
AutoveloxProject/
├── Autovelox.Data/
│   ├── Dipendenze/
│   └── Models/
│       ├── AutoveloxContext.cs
│       ├── Comune.cs
│       ├── Mappa.cs
│       ├── Provincium.cs
│       ├── Regione.cs
│       └── RipartizioneGeografica.cs
└── Autovelox.WebAPI/
    ├── Connected Services/
    ├── Dipendenze/
    ├── Properties/
    ├── Controllers/
    │   ├── ComuniController.cs
    │   ├── MappeController.cs
    │   ├── ProvinceController.cs
    │   ├── RegioniController.cs
    │   └── RipartizioniGeograficheController.cs
    ├── appsettings.json
    ├── Autovelox.WebAPI.http
    └── Program.cs
```

---

*Guida generata il 24/05/2026 — AutoveloxProject su SQL Server 2025 + .NET 10 + Visual Studio 2026*
