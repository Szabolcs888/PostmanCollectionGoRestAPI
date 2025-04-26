# Go REST API Test Collection

<br>

<table>
<tr>
<td>
<a href="https://www.youtube.com/watch?v=8dA4x-k2LUY" style="display: block;">
<img src="https://github.com/user-attachments/assets/c5880b11-f9cc-4539-91a9-de0b1f56d8a3" alt="Video 1" width="300" />
   </a>
<br>
<p align="center">Collection demo video</p>
</td>
<td>
<img src="https://github.com/user-attachments/assets/d6d9b6b2-19f2-43df-9f0e-21d721a23eaf" alt="Kép 2" width="300"/>
<br>
<p align="center">Tests example screenshot</p>
</td>
<td>
<img src="https://github.com/user-attachments/assets/af9fdcba-d228-432e-863f-6f396ecab974" alt="Kép 2" width="300"/>
<br>
<p align="center">HTML Report screenshot</p>
</td>
</tr>
</table>

## Overview

This Postman collection provides a comprehensive automated test suite for the GoREST API (v2 - `https://gorest.co.in/public/v2/`). It covers the full CRUD operations for the main resources (Users, Posts, Comments, Todos), and also includes comprehensive error handling scenarios/tests and authentication tests.

## Prerequisites

1.  **Postman** and/or **Newman**
2.  **Valid API Token:** You need a valid Bearer token from GoREST. Set this token in the `token` collection variable. The collection includes a default token, but it might expire or become invalid.

## Collection Structure and Section Purposes

The collection is organized into logical folders. It's important to note that the collection contains descriptions (`description`) at the folder level and also at the individual request level, **as well as tests at certain folder levels (e.g., subfolders of 'Authentication', 'Error Handling')**. The latter allows for common checks (e.g., status code, basic response structure) to be centrally defined and automatically run for all requests within that folder, following the DRY principle and keeping the tests for individual requests clean.

The purposes of each main section are as follows:
1.  **Authentication:**
    *   **Purpose:** To verify that the API correctly handles authentication using Bearer tokens.
    *   **Tests:** Includes checks for requests sent without an authorization token (`missingAuthorizationToken401`) and with an invalid token (`invalidToken401`), expecting `401 Unauthorized` responses with specific error messages.

2.  **Users CRUD:**
    *   **Purpose:** To test the full lifecycle of a User resource.
    *   **Workflow:** Creates a user (and saves the `userId`), updates it using both PUT and PATCH methods, retrieves the user by ID, retrieves the list of all users (verifying the created user is included), finally deletes the user (at the end of the collection run).

3.  **Posts CRUD:**
    *   **Purpose:** To test the full lifecycle of a Post resource, associated with a user.
    *   **Workflow:** Creates a post (and saves the `postId`) associated with the previously created user (`userId`), updates it (PUT/PATCH), retrieves it by `postId`, retrieves all posts for the given user, retrieves all public posts, finally deletes the post (at the end of the collection run).

4.  **Comments CRUD:**
    *   **Purpose:** To test the full lifecycle of a Comment resource, associated with a post.
    *   **Workflow:** Creates a comment (and saves the `commentId`) associated with the previously created post (`postId`), updates it (PUT/PATCH), retrieves it by `commentId`, retrieves all comments for the given post, retrieves all public comments, finally deletes the comment (at the end of the collection run).

5.  **Todos CRUD:**
    *   **Purpose:** To test the full lifecycle of a Todo resource, associated with a user.
    *   **Workflow:** Creates a todo (and saves the `todoId`) associated with the user (`userId`), updates it (PUT/PATCH), retrieves it by `todoId`, retrieves all todos for the given user, retrieves all public todos, finally deletes the todo (at the end of the collection run).

6.  **Error Handling:**
    *   **Purpose:** To verify the API's robustness by testing how it handles invalid requests and edge cases.
    *   **Tests:** Includes attempts to:
        *   Perform GET, PUT, PATCH, DELETE operations on non-existent resources (expected result: `404 Not Found`).
        *   Create resources with duplicate data (e.g., existing email - expected result: `422 Unprocessable Entity`).
        *   Create resources with missing or incomplete required fields (expected result: `422 Unprocessable Entity`).

## Running the Collection

It is **recommended** to run the collection using the **Postman Collection Runner** or **Newman**, as the tests depend on data created in previous steps (`userId`, `postId`, `commentId`, `todoId`).

**Reason for Sequential Execution:** The GoREST API operates with live, public, and mutable data used by others, so the data changes constantly. Consequently, creating static, independent tests that rely on fixed resource identifiers (e.g., assuming user `7855260` always exists with specific data) is unreliable and prone to failure. This dynamic nature necessitates the sequential workflow employed here, where resources are created, their IDs are dynamically saved, tested, and finally deleted.

*   **Execution Order:** The tests follow a defined workflow (Authentication -> User -> Post -> Comment -> Todo -> Error Handling -> Cleanup).
*   **Dynamic ID Handling:** The `createUser`, `createPost`, `createComment`, and `createTodo` requests capture the ID of the newly created resource and store it in the corresponding collection variable (`userId`, `postId`, etc.).
*   **Execution Flow Control:** If any "create" request fails (e.g., does not return a `201 Created` status with a valid ID), the test script uses the `pm.execution.setNextRequest(null)` command. **This intentionally stops the Collection Runner** to prevent subsequent tests from failing due to missing prerequisite data (like a `userId`).

## Key Features and Observations

*   **Comprehensive CRUD:** Covers all basic operations for the main API resources.
*   **Sequential Workflow:** Demonstrates testing of complex, interdependent API workflows.
*   **Two-Tier Response Time Check:** A global test script checks response times:
    *   Expects responses to be under `500ms`.
    *   If a response exceeds `500ms`, it logs a warning (`console.warn`), but the test still passes if the time is under `1500ms`.
    *   Responses exceeding `1500ms` will fail the test. This provides flexibility while still monitoring performance.
*   **PUT Method Behavior:** It's observed that the API's PUT endpoints (`/users/{id}`, `/posts/{id}`, etc.) behave more like the PATCH method. They update the fields provided in the request body but **do not remove or nullify fields that are omitted** from the request. This differs from the standard HTTP specification, where PUT is expected to replace the entire resource.
*   **DELETE Status Codes:** Tests for DELETE operations (`deleteUser`, `deletePost`, etc.) accept both `204 No Content` and `200 OK` (with an empty body) status codes, accommodating potential variations in the API's responses.
*   **Other Information:** Further details can be found in the 'Description' sections of specific collection folders and requests.

## Running with Newman

Newman is the command-line runner for Postman.

1.  **Export:**
    Export the collection (`GoRESTAPITestCollection.json`), ensuring your `token` is included.
3.  **Basic Run:**
    ```bash
    cd c:\collection path
    newman run GoRESTAPITestCollection.json
    ```
4.  **Run with HTML Report (using htmlextra reporter):**
    *   Install the reporter:
    ```bash
    npm install -g newman-reporter-htmlextra
    ```
    *   Run Newman with the reporter:
    ```bash
    newman run GoRESTAPITestCollection.json -r htmlextra
    ```
    *   This will generate a detailed HTML report in a newly created `newman` folder.

## Variables

Key collection variables used:

*   `baseUrl`: The base URL of the API (e.g., `https://gorest.co.in`).
*   `token`: **(Required)** Your valid GoREST API Bearer token.
*   `invalidToken`: An intentionally invalid token for authentication tests.
*   `userId`, `postId`, `commentId`, `todoId`: Dynamically populated by the create requests during the run.
*   `userEmail`, `userName`, `modifiedUserName`: Test data used for creating/updating users. The `userEmail` must be unique on the first run.
*   `invalidUserId`, `invalidPostId`, `invalidCommentId`, `invalidTodoId`: Non-existent IDs for 404 error handling tests.

#
#
<br>

# Go REST API Test Collection

## Áttekintés

Ez a Postman collection egy átfogó automatizált tesztkészletet biztosít a GoREST API-hoz (v2 - `https://gorest.co.in/public/v2/`). Lefedi a fő erőforrások (Users, Posts, Comments, Todos) teljes CRUD műveleteit, valamint átfogó hibakezelési forgatókönyveket/teszteket és authentikációs teszteket is tartalmaz.

## Előfeltételek

1.  **Postman** és/vagy **Newman** 
2.  **Érvényes API Token:** Szükséged van egy érvényes Bearer tokenre a GoREST-től. Állítsd be ezt a tokent a `token` collection változóban. A collection tartalmaz egy alapértelmezett tokent, de az lejárhat vagy érvénytelenné válhat.

## A Collection Szerkezete és a szekciók céljai

A collection logikai mappákba van szervezve. Fontos megjegyezni, hogy a collection mappaszinten, illetve egyes requestek szintjén is tartalmaz leírásokat (`description`), **valamint bizonyos mappák szintjén (pl. 'Authentication', 'Error Handling' almappái) teszteket is**. Ez utóbbi lehetővé teszi a közös ellenőrzések (pl. státuszkód, alapvető válaszstruktúra) központi definiálását és automatikus lefuttatását az adott mappa alá tartozó összes requestre, követve a DRY elvet és tisztán tartva az egyedi requestek tesztjeit.

Az egyes fő szekciók céljai a következők:
1.  **Authentikáció (Authentication):**
    *   **Cél:** Annak ellenőrzése, hogy az API helyesen kezeli-e az authentikációt Bearer tokenek segítségével.
    *   **Tesztek:** Tartalmaz ellenőrzéseket az authorizációs token nélkül (`missingAuthorizationToken401`) és érvénytelen tokennel (`invalidToken401`) küldött kérésekre, `401 Unauthorized` válaszokat várva specifikus hibaüzenetekkel.

2.  **Felhasználók CRUD (Users CRUD):**
    *   **Cél:** Egy User resource teljes életciklusának tesztelése.
    *   **Munkafolyamat:** Létrehoz egy felhasználót (és elmenti a `userId`-t), frissíti azt PUT és PATCH metódusokkal is, lekérdezi a felhasználót azonosító alapján, lekérdezi az összes felhasználó listáját (ellenőrizve, hogy a létrehozott felhasználó szerepel-e benne), végül törli a felhasználót (a collection futásának végén). 

3.  **Bejegyzések CRUD (Posts CRUD):**
    *   **Cél:** Egy Post resource teljes életciklusának tesztelése, egy user-hez kapcsolva.
    *   **Munkafolyamat:** Létrehoz egy bejegyzést (és elmenti a `postId`-t) az előzőleg létrehozott felhasználóhoz (`userId`), frissíti azt (PUT/PATCH), lekérdezi `postId` alapján, lekérdezi az adott felhasználó összes bejegyzését, lekérdezi az összes nyilvános bejegyzést, végül törli a bejegyzést (a collection futásának végén).

4.  **Kommentek CRUD (Comments CRUD):**
    *   **Cél:** Egy Comment resource teljes életciklusának tesztelése, egy post-hoz kapcsolva.
    *   **Munkafolyamat:** Létrehoz egy kommentet (és elmenti a `commentId`-t) az előzőleg létrehozott bejegyzéshez (`postId`), frissíti azt (PUT/PATCH), lekérdezi `commentId` alapján, lekérdezi az adott bejegyzéshez tartozó összes kommentet, lekérdezi az összes nyilvános kommentet, végül törli a kommentet (a collection futásának végén). 

5.  **Teendők CRUD (Todos CRUD):**
    *   **Cél:** Egy Todo resource teljes életciklusának tesztelése, egy user-hez kapcsolva.
    *   **Munkafolyamat:** Létrehoz egy teendőt (és elmenti a `todoId`-t) a felhasználóhoz (`userId`), frissíti azt (PUT/PATCH), lekérdezi `todoId` alapján, lekérdezi az adott felhasználó összes teendőjét, lekérdezi az összes nyilvános teendőt, végül törli a teendőt (a collection futásának végén). 

6.  **Hibakezelés (Error Handling):**
    *   **Cél:** Az API robusztusságának ellenőrzése azáltal, hogy teszteljük, hogyan kezeli az érvénytelen kéréseket és a szélsőséges eseteket.
    *   **Tesztek:** Kísérleteket tartalmaz a következőkre:
        *   Nem létező erőforrás GET, PUT, PATCH, DELETE műveletei (várt eredmény: `404 Not Found`).
        *   Erőforrások létrehozása duplikált adatokkal (pl. már létező email - várt eredmény: `422 Unprocessable Entity`).
        *   Erőforrások létrehozása hiányos vagy kötelező mezők nélküli adatokkal (várt eredmény: `422 Unprocessable Entity`).

## A Collection Futtatása

A collectiont **ajánlott** a **Postman Collection Runner** vagy a **Newman** segítségével futtatni, mivel a tesztek az előző lépésekben létrehozott adatoktól (`userId`, `postId`, `commentId`, `todoId`) függenek. 

**A Szekvenciális Futtatás Oka:** A GoREST API élő, nyilvános és módosítható adatokkal működik, amelyet mások is használnak, ezért az adatok folyamatosan változnak. Ebből következően statikus, független tesztek létrehozása, amelyek fix erőforrásazonosítókra (pl. annak feltételezése, hogy a `7855260`-as felhasználó mindig létezik specifikus adatokkal) támaszkodnak, megbízhatatlan és hibára hajlamos. Ez a dinamikus természet teszi szükségessé az itt alkalmazott szekvenciális munkafolyamatot, ahol az erőforrásokat létrehozzuk, azonosítóikat dinamikusan elmentjük, teszteljük, és végül töröljük.

*   **Futtatási Sorrend:** A tesztek egy meghatározott munkafolyamatot követnek (Authentication -> User -> Post -> Comment -> Todo -> Error Handling -> Cleanup). 
*   **Dinamikus ID Kezelés:** A `createUser`, `createPost`, `createComment` és `createTodo` kérések rögzítik az újonnan létrehozott erőforrás azonosítóját, és eltárolják azt a megfelelő collection változóban (`userId`, `postId`, stb.).
*   **Futtatási Folyamat Vezérlése:** Ha bármelyik "create" kérés sikertelen (pl. nem `201 Created` státusszal és érvényes ID-vel tér vissza), a testscript a `pm.execution.setNextRequest(null)` parancsot használja. **Ez szándékosan leállítja a Collection Runnert**, hogy megakadályozza a későbbi tesztek meghiúsulását a hiányzó előfeltétel-adatok (mint pl. egy `userId`) miatt.

## Kulcsfontosságú Jellemzők és Megfigyelések

*   **Átfogó CRUD:** Lefedi a fő API erőforrások összes alapvető műveletét.
*   **Szekvenciális Munkafolyamat:** Bemutatja komplex, egymástól függő API munkafolyamatok tesztelését.
*   **Kétlépcsős Válaszidő Ellenőrzés:** Egy globális testscript ellenőrzi a válaszidőket:
    *   Elvárja, hogy a válaszok `500ms` alatt legyenek.
    *   Ha egy válasz meghaladja az `500ms`-t, figyelmeztetést naplóz (`console.warn`), de a teszt még sikeres, ha az idő `1500ms` alatt van.
    *   Az `1500ms`-t meghaladó válaszok esetén a teszt sikertelen lesz. Ez rugalmasságot biztosít, miközben továbbra is figyeli a teljesítményt.
*   **PUT Metódus Viselkedése:** Megfigyelhető, hogy az API PUT végpontjai (`/users/{id}`, `/posts/{id}`, stb.) inkább a PATCH metódushoz hasonlóan viselkednek. Frissítik a kérés törzsében megadott mezőket, de **nem távolítják el vagy nullázzák ki azokat a mezőket, amelyek kimaradnak** a kérésből. Ez eltér a standard HTTP specifikációtól, ahol a PUT-tól az egész erőforrás lecserélését várnánk.
*   **DELETE Státuszkódok:** A DELETE műveletek tesztjei (`deleteUser`, `deletePost`, stb.) elfogadják mind a `204 No Content`, mind a `200 OK` (üres törzzsel) státuszkódot, alkalmazkodva az API válaszainak lehetséges eltéréseihez.
*   **Egyéb információk:** Egyéb információk találhatóak még a collection adott mappáinak és requestjeinek 'Description" szekcióiban.

## Futtatás Newmannel

A Newman a Postman parancssori futtatója.

1.  **Exportálás:**
    Exportáld a collectiont (`GoRESTAPITestCollection.json`), ügyelve arra, hogy a `token`-ed is benne legyen.
3.  **Alap Futtatás:**
    ```bash
    cd c:\collection path
    newman run GoRESTAPITestCollection.json
    ```
4.  **Futtatás HTML Riporttal (htmlextra reporter használatával):**
    *   Telepítsd a reportert:
    ```bash
    npm install -g newman-reporter-htmlextra
    ```
    *   Futtasd a Newmant a reporterrel:
    ```bash
    newman run GoRESTAPITestCollection.json -r htmlextra
    ```
    *   Ez létrehoz egy részletes HTML riportot egy újonnan létrejövő newman mappába.

## Változók

Kulcsfontosságú használt collection változók:

*   `baseUrl`: Az API alap URL-je (pl. `https://gorest.co.in`).
*   `token`: **(Kötelező)** Az érvényes GoREST API Bearer tokened.
*   `invalidToken`: Egy szándékosan érvénytelen token az authentikációs tesztekhez.
*   `userId`, `postId`, `commentId`, `todoId`: Dinamikusan töltődnek fel a létrehozó kérések által a futás során.
*   `userEmail`, `userName`, `modifiedUserName`: Felhasználók létrehozásához/frissítéséhez használt tesztadatok. Az `userEmail`-nek egyedinek kell lennie az első futtatáskor.
*   `invalidUserId`, `invalidPostId`, `invalidCommentId`, `invalidTodoId`: Nem létező azonosítók a 404-es hibakezelési tesztekhez.
