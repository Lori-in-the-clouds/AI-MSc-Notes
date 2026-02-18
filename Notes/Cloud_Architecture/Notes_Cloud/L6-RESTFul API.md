# L6-RESTFul API

Number: 3
Status: Check phase
Type: lab

# 1. Introduzione

<aside>
💡

Le **RESTful API** (Representational State Transfer) sono un'architettura software utilizzata per definire interfacce che permettono la comunicazione asincrona e la manipolazione di risorse (oggetti) tramite operazioni **CRUD** (Create, Read, Update, Delete).

L'obiettivo è standardizzare lo scambio di dati, identificando le risorse in modo univoco tramite **HTTP URI** e determinando l'azione da compiere tramite i **Metodi HTTP**.

</aside>

Possono essere progettate per due principali tipi di interazione:

- **Human–Machine Interface** → l’API viene consumata da un frontend destinato agli utenti finali, tipicamente generato tramite **HTML**.
- **Machine–Machine Interface** → l’API viene utilizzata da altri servizi o applicazioni, senza la necessità di un frontend HTML.

Una caratteristica fondamentale delle RESTful API è il modello **stateless**: ogni richiesta HTTP è indipendente dalle precedenti e contiene tutte le informazioni necessarie per essere elaborata dal server. Lo sviluppo di una RESTful API si basa sulla mappatura delle **operazioni CRUD** sui metodi HTTP standard:

- **Create** → `POST`
- **Read** → `GET`
- **Update** → `PUT o PATCH`
- **Delete** → `DELETE`

Questo approccio consente di creare interfacce **semplici**, **scalabili** e facilmente **integrabili** tra sistemi diversi.

---

# 2. Standard API

Quando si sviluppano API REST, è buona norma seguire standard consolidati in modo da rendere l’interfaccia coerente, leggibile e facilmente utilizzabile da client diversi.

- **Struttura dell’URL** → un URL tipico segue una struttura ben definita: `/api/<version>/<resource>/<identifier>`.
    
    Dove:
    
    - `api` → indica che si tratta di un’API
    - `version` → versione dell’API, fondamentale per l’evoluzione del software, poiché consente di introdurre modifiche mantenendo la **retrocompatibilità** con i client che utilizzano versioni precedenti.
    - `resource` → risorsa esposta dall’API (es. colors)
    - `identifier` → identificatore della risorsa (es. nome o ID)
    
    In uno stile REST corretto, **le azioni non sono espresse nell’URL**, ma sono determinate dal **metodo HTTP** utilizzato (GET, POST, PUT, DELETE).
    
- **Serializzazione dei dati** → durante la comunicazione client–server, i dati devono essere **serializzati**. Per la serializzazione si utilizza il **JSON schema**.
    
    ![image.png](L6-RESTFul%20API/image.png)
    

## 2.1. Codice

- **Per creare un’ API RESTFUL partendo da un’app flask bisogna wrapparla con la libreria `flask_restful`:**
    
    ```python
    from flask_restful import Resource, Api
    
    app = Flask(__name__, static_url_path='/static',static_folder='static')
    api=Api(app)
    
    if __name__ == '__main__':
        app.run(host='127.0.0.1', port=8080, debug=True)
    ```
    
- **Invece di usare le route classiche (`@app.route`), qui le azioni sono raggruppate in classi che ereditano da Resource:** l’oggetto resource permette di eseguire il mapping tra metodi della classe (get, post, put, delete) e verbi HTTP. Bisogna solo associare l’URL base alla classe, registrando la risorsa nel seguente modo:
    
    ```python
    basePath = '/api/v1'
    api.add_resource(ColorsResource, f'{basePath}/colors/<string:colorname>')
    ```
    
    Se nell’URL base della risorsa ci sono dei parametri che si vogliono passare ai metodi si utilizza la seguente sintassi: `<tipo_di_dato:nome_dato>`.
    
- **Metodi get e delete:**
    - non accettano un body (non lo controllano).
    - devono ritornare dati e codice di risposta, oppure solo codice di risposta.
    
    ```python
    def get(self, colorname):
            colorname=colorname.lower()
            c=colors_dao.get_color_by_name(colorname)
            if c is None:
                return None, 404
            else:
                return c, 200
    ```
    
- **Metodi put e post:**
    - accettano un body e devono serializzare i dati in json e validarli con funzione scritta in python (guarda `validate_colordata()`), senza affidarsi a validators come con WTForm.
    - devono ritornare solo codice di risposta.
    
    ```python
    def post(self, colorname):
            colordata=request.json
            print('got POST request' + str(colordata))
            if not self.validate_colordata(colordata):
                return None, 400
            if colors_dao.get_color_by_name(colorname) is not None:
                return None, 409
    				colors_dao.add_color(colorname, colordata['red'], colordata['green'], colordata['blue'])
    				return None, 201	
    				
    def validate_colordata(self, colordata):
            for k in ['red', 'green', 'blue']:
                if k not in colordata.keys():
                    return False
                if not isinstance(colordata[k], (int)):
                    return False
                if colordata[k]<0 or colordata[k]>255:
                    return False
            return True
    ```
    

---

# 3. OpenAPI e Swagger

**OpenAPI** è uno **standard descrittivo** per documentare una RESTful API. La descrizione dell’API è contenuta in un file **JSON o YAML**, che specifica in modo formale:

- **Endpoint** disponibili e operazioni supportate (GET, POST, PUT, DELETE, …)
- **Parametri di input** e **output** per ogni operazione
- **Codici di risposta HTTP**
- **Schemi di autenticazione**
- **Modelli dei dati** scambiati tra client e server

Il file OpenAPI rappresenta quindi un **contratto** tra chi fornisce l’API e chi la utilizza.

<aside>
📌

**Swagger**

Swagger è una libreria che, partendo da un file OpenAPI (JSON o YAML), genera un’interfaccia grafica per visualizzare la documentazione dell’API e per testare direttamente gli endpoint.

</aside>

## 3.1. Struttura di un file OpenAPI **(Swagger 2.0)**

```yaml
#Per prima cosa si indica la versione di Swagger da utilizzare (la 2.0 è la più diffusa), quindi si forniscono le informazioni principali dell’API, come titolo, descrizione e versione.
swagger: "2.0"
info:
  title: "Colors"
  description: "Manage color with RGB components"
  version: "1.0.0"

#Si specifica l'url sulla cui è esposta l'API
host: "api-dot-colors20221001.appspot.com"

#Si include poi ilprefisso comune a tutti gli endpoint (include la versione) e i protocolli supportati
basePath: "/api/v1"
schemes:
  - "https"

#Le richieste associate a questa risorsa accettano un parametro 
#La proprietà in specifica dove si trova il parametro: può essere nel percorso (path), nell’intestazione (header), nel corpo della richiesta (body) o nei parametri di query (query). 
paths:
  /colors/{colorName}:
    parameters: 
      - name: colorName
        in: path
        description: "Name of the color"
        type: string
        required: true

		#Ogni metodo deve definire le risposte possibili.Nel caso del metodo GET, viene restituito sia un codice di stato sia dei dati; 
		#la struttura dei dati restituiti è specificata tramite $ref: "#/definitions/Color", che fa riferimento allo schema definito nella sezione definitions.
    get:
      description: "Get color name and RGB from color name"
      operationId: "colorGet"
      responses:
        200:
          description: "Success."
          schema:
            $ref: "#/definitions/Color"
        404:
          description: "Color not found."

		#Metodo POST 
    post:
      description: "Add a new color"
      operationId: "colorPost"
      consumes: #indica il tipo di contenuto che il server si aspetta nel corpo della richiesta.
        - application/json #significa che il POST deve inviare dati in formato JSON.
      parameters:
        - in: body #il parametro si trova nel corpo della richiesta
          name: color #nome del parametro
          description: "Color object"
          schema:
            $ref: "#/definitions/Color" #definisce la struttura dei dati attesi
          required: true
      responses:
        201:
          description: "Color created."
        400:
          description: "Invalid color."
        409:
          description: "Conflict. Color already exists."

#In fondo al file vi sono le definizioni delle strutture/schemi dei dati
definitions:
  ColorList:
    type: array
    items:
      type: string

  Color:
    type: object
    properties:
      red:
        type: integer
        minimum: 0
        maximum: 255
      green:
        type: integer
        minimum: 0
        maximum: 255
      blue:
        type: integer
        minimum: 0
        maximum: 255
```

<aside>
📌

**Riassunto Parametri:**

Elementi base del file OpenAPI:

- **swagger**
- **info**
- **host**
- **basePath**
- **schemes**
- **paths**
- **definitions**

Elementi base di un metodo

- **operationId**
- **description**
- **consumes** / **produces**: un metodo può consumare dei parametri o produrne, con queste chiavi si specifica il formato dei dati
    
    ![Screenshot 2026-01-04 at 11.15.33.png](L6-RESTFul%20API/Screenshot_2026-01-04_at_11.15.33.png)
    
- **parameters**
- **responses**
</aside>

## 3.2. WTForms

WTForms è una **libreria Python** che permette di definire **oggetti Form** con i campi desiderati.

1. **Definizione del form:**
    
    ```python
    from wtforms import Form, IntegerField, SubmitField, validators
    
    class ColorForm(Form):
        red =   IntegerField('Red', [validators.NumberRange(min=0, max=255)])
        green = IntegerField('Green', [validators.NumberRange(min=0, max=255)])
        blue =  IntegerField('Blue', [validators.NumberRange(min=0, max=255)])
        submit= SubmitField('Submit')
    ```
    
    Ogni campo può avere una lista di **validators**, che controllano la validità dei dati **lato backend**.
    
2. **Creazione dell’istanza del form e binding con dati esistenti:**
    - Se si vuole popolare il form con i dati inviati dall’utente:
        
        ```python
        cform = ColorForm(request.form)
        ```
        
    - Se si vuole popolare il form con un oggetto Python o un dizionario (ad esempio dati dal database):
        
        ```python
        c = {'red': 0, 'green': 0, 'blue': 0}
        cform = ColorForm(obj=Struct(**c))
        ```
        
        Questo permette al form di avere come valore iniziale i dati presenti nell’oggetto c.
        
3. **Validazione dei dati lato backend:**
    
    ```python
    if cform.validate(): #esegue tutti i validators associati ai campi.
        colors_dao.add_color(color, cform.red.data, cform.green.data, cform.blue.data)
    else:
        print(cform.errors) #Gli errori vengono raccolti in cform.errors
    ```
    
4. **Rendering automatico nel template HTML con Jinja2:** quando passi il form a un template **Jinja2**, WTForms genera automaticamente il codice HTML per ciascun campo, applicando anche gli attributi HTML dei validator **di default** (come `min`, `max` per un `IntegerField`).
    
    ```html
    <form method="POST">
        <table>
            <tr><td>{{ color_form.red.label }}</td><td>{{ color_form.red }}</td></tr>
            <tr><td>{{ color_form.green.label }}</td><td>{{ color_form.green }}</td></tr>
            <tr><td>{{ color_form.blue.label }}</td><td>{{ color_form.blue }}</td></tr>
            <tr><td></td><td>{{ color_form.submit }}</td></tr>
        </table>
    </form>
    ```
    
    **N.B.** WTForms applica automaticamente solo i validators di default (come `NumberRange`, `DataRequired`). Se scrivi validator custom, devi gestire eventualmente anche il controllo lato frontend con JavaScript.
    
5. **Per passare un form al template HTML:**
    
    ```python
    cform = ColorForm(obj=Struct(**c))
    return render_template('color.html', color_form=cform)
    ```
    

---