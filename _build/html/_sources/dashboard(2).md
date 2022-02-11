# Dashboard

:::{note}
Hier wird erläutert, wie das Dashboard aufgebaut ist, welche Technologien wir verwendet haben und wie wir das Ganze technisch umgesetzt haben. Das Ganze wird mit Codebeispielen dargestellt. 

Die Codebeispiele können dabei leicht abweichen, um es für den Leser besser verständlich zu machen.
:::

## Genereller Aufbau

Das Dashboard besteht aus mehreren Tabs, die unterschiedliche Themen behandeln. Dabei gehen wir Thematisch vor. Zunächst kann man sich im Tab "Schaden durch Hacks" einen ersten Überblick verschaffen, was für Schäden Cyber Attacken im Allgemeinen nach sich ziehen. Um sich weiter zu informieren, wird im Tab "Hackermethoden" genauer darauf eingegangen, wie Hacks durchgeführt werden und welche Arten am meisten verbreitet sind. Die zweit häufigsten Angriffe, mit der Schwachstelle Mensch sind Phishing Attacken. Deswegen kann sich der Nutzer im Tab "Phishing" über mögliche Absichten von solchen Mails informieren und auch schauen, welche Branchen und Abteilungen häufig betroffen sind. Im letzten Tab "Passwortsicherheit" wird, wie der Name schon sagt, noch auf die Passwortsicherheit ein eingegangen. Hier kann der Nutzer Passwörter testen und schauen, ob sich vielleicht eins der eigen verwendeten Passwörter in der Wordcloud der meist verwendeten Passwörter wiederfindet.

Insgesamt haben wir bei den einzelnen Tabs darauf geachtet, dass der Nutzer sich beim lesen bzw. anschauen, von links nach rechts durcharbeiten kann und möglichst alle Informationen auf einer Seite (also ohne Scrollen) hat. Die Leserichtung von links nach rechts haben wir gewählt, da diese in europäischen Raum so üblich ist und uns am intuitivsten erschien. Haben die Diagramme nicht auf eine Seite gepasst, haben wir sie so angeordnet, dass sich von oben nach unten durchgearbeitet werden kann. Auch dies ist ist der Lesefluss, denn man in europäischen Raum von Büchern etc. gewöhnt ist.

## Farbauswahl

Die Farben haben wir aus unserer Farbpalette übernommen.
Wir haben uns für ein dunklen Hintergrund entschieden, da dies oft angenehmer zum anschauen für das menschliche Auge ist. Die Daten haben wir in einem hellen Blauton dargestellt, um sie vom dunklen Hintergrund abzuheben und gut sichtbar zu machen. Akzente haben wir mit einem dunklen Lila gesetzt. Dadurch wollen wir die Aufmerksamkeit des Nutzers lenken. Es ist auffallend genug, aber nicht so aggressiv wie ein rot. 

## Verwendete Technologien

### Applikation

Das Dashboard haben wir mit Dash umgesetzt. Die Applikation zu erstellen und zu starten war dann relativ einfach:

```
import dash
app = dash.Dash(__name__)


if __name__ == '__main__':
    app.run_server(debug=True)

```

Für die Gestaltung des Dashboards haben wir eine app.css Datei erstellt, die wir im Ordner assets hinterlegt haben. Hier wurde das Styling des Dashboards definiert wie beispielsweise die Hintergrundfarbe sowie das Aussehen der Überschriften:

```
body {
    font-family: sans-serif;
    background-color: #072542;
    color: #FFFFFF;
}
h1, h2, h3, h4, h5, h6 {
    color: #FFFFFF;
    text-align: center;
}
```

### Datenhandling

Die Daten haben, die wir verwenden, liegen uns zu Beginn in Excel-Dateien vor. Mit dem Framework Pandas lesen wir diese Daten ein und bearbeiten sie. Pandas verwenden wir als zentrales Tool, um Daten zu bearbeiten und die Daten am Ende für den Export/ Download bereitzustellen. 

```
self.df = pd.read_excel("./datasets/DataBreaches.xlsx")

self.df = self.df.rename(columns= {'year   ':'year'}) # Spalte year umbennenen (aufgrund von Leerzeichen nach "year")
self.df = self.df.drop(self.df.index[[0]]) # Löschen der ersten Zeile
self.df["year"] = self.df["year"].astype("int") # Umwandlung des Datentyps von Year von Object in Integer mit der typecasting_from_column-Methode
self.df["records lost"] = pd.to_numeric(self.df["records lost"]) # Umwandlung des Datentyps von records lost von Object in numeric
self.df["organisation"] = self.df["organisation"].astype("string") # Umwandlung des Datentyps von Organisation von Object in string
self.df['organisation_name'] = ''
```

Das Dataframe kann dann in der ganzen Klasse verwendet werden. Zum einen um Diagramme zu erstellen und auch für den Download der Nutzer auf dem Dashboard.

Das Framework ist dabei sehr tiefgehend und bietet viele Funktionen. Leider konnten wir in dieser kurzen Zeit nicht alle Funktionen verstehen oder noch nicht richtig anwenden. Deswegen mussten wir auch häufig Workarounds für bestimmte Probleme finden. Zum Beispiel wollten wir einmal die Zeitpunkte von Databreaches gleichmäßig im Monat verteilen, sodass im Scatter-Plot-Diagramm die Daten nicht übereinander liegen. Dafür haben wir jeden Datenpunkt noch ein Variable in der Spanne von 0 bis 30 zugewiesen, um die Verteilung zu simulieren. Je nachdem, wie viele Daten in einem Monat gibt, waren die Datenpunkte dann entsprechend weit auseinander verteilt.

```
for year in range( 2004, 2022):
    # Herausfiltern der Firmennamen, bei denen die Schaden zu den größten 10% gehören, damit kleinere Punkte keine Beschriftung haben
    self.df['organisation_name'] = np.where((self.df['year'] == year) & (self.df['records lost'] >= self.df['records lost'].max()*0.1), self.df['organisation'], self.df['organisation_name'])
    
    # gleichmäßiges Verteilen der einzelnen Vorfälle auf einen Monat
    for month in range(1, 13):
        date = str(year) + '-' + str(month).zfill(2) + '-01'

        new_df = self.df.loc[(self.df["date"] == date)]

        step = 30//(new_df.shape[0]+1) # Stepgröße um zu berechnen, in welchem Abstand die Vorfälle platziert werden
        start = 0
        for index, row in new_df.iterrows():
            self.df.loc[index, 'date'] = '%s-%s-%s' % (year, str(month).zfill(2), str(start+step).zfill(2))
            start += step
```

### Diagramme

Bei der Auswahl der Diagramme haben wir Data to Viz (https://www.data-to-viz.com/), als Hilfestellung verwendet. Dort haben wir ausgewählt, welche Art von Variablen wir in unseren Daten haben (numerisch oder kategorisch). Je nachdem, wie viele numerische oder kategorische Variablen wir hatten, haben wir uns in der Baumstruktur der Webseite durchgearbeitet und uns dann eine Diagramm-Art ausgesucht.
Die Diagramme haben wir zum großen Teil mit Plotly Express erstellt. Die Diagramme konnten dadurch im ersten Schritt schnell aus den vorhandenen Daten erstellt werden. 

```
fig = px.line(
    self.df_fig1, 
    x="year", 
    y="records lost", 
    labels={
        "year": "",
        "records lost": "entstandener Schaden (in Mrd. US$)",
    },
    title='', 
    markers=True
)

```
Die Anpassung und der feinschliff war dafür leider etwas schwieriger und teilweise konnten wir bestimmte Vorhaben auch nicht umsetzen. 

```
fig.update_xaxes(showgrid=False, title_font_family="Arial", title_font_color=color)
        fig.update_yaxes(showgrid=False, title_font_family="Arial", title_font_color=color)
        fig.update_layout(

            # Anpassen des Titels
            title={
                'text': "Verlauf des enstandenen Schadens durch Datenlecks",
                # Titel linksbündig an die y-Achse anpassen
                'y':0.87, # Fehleranfällig, da die Größe des Diagramms über die Position der Überschrift bestimmt
                'x':0.0,
                'xref': "paper",
                'xanchor': 'left',
                'yanchor': 'top'
            },

            # Hier wird der Hintergrund transparent gemacht und die Schriftfarbe je nach Darkmoder bestimmt (siehe Report/Allgemein/Diagramm Anpassungen)
            plot_bgcolor = "rgba(0,0,0,0)",
            paper_bgcolor = "rgba(0,0,0,0)",
            font_color=color,

            # Anpassen der X- und Y-Achse
            xaxis= dict(
                range=[self.df['year'].max() - 7 - 0.5, self.df['year'].max() + 0.5],
                dtick= 1,
                ticks = "outside",
                tickwidth = 1,
                tickcolor = color,
                ticklen = 8,
                tickfont = dict(family = 'Arial', size = 14),
                showline = not darkmode,
                linewidth = 1,
                linecolor = color,
            ),

            yaxis = dict(
                range=[0, df_fig1['records lost'].max()*1.2],
                ticks = "outside",
                tickwidth = 1,
                tickcolor = color,
                ticklen = 8,
                ticklabelposition="outside",
                showline = True,
                linewidth = 1,
                linecolor = color,
            ),
        )
        
        # Anpassen der Farbe der Linien
        fig.update_traces(
            marker = dict(
                color = '#4DDBE3',
                size = 10,
                opacity = 0.8,
            ),
            line = dict(
                color = '#4DDBE3',
                width = 2
            ),
        )
```


## Ordnerstruktur und Technischer Aufbau

Die Struktur haben wir so aufgebaut, dass wir für jeden Tab eine eigene Klasse erstellt haben. Jede Klasse bietet dabei eine get_layout() Funktion, die das Dash Layout zurückgibt. Durch das aufteilen konnte sowohl Hauptdatei deutlich übersichtlicher gestaltet werden und man wusste direkt, wo man suchen sollte, wenn ein Fehler auftritt. Wir haben uns auch dafür entschieden, die einzelnen Diagramme für die Seiten ebenfalls in eigenen Klassen zu verwalten. Dadurch wurde die Logik von Diagramm erstellen und Dashboard sauber voneinander getrennt.

**Problem**

Eigentlich wollten wir auch die Funktionen für die einzelnen Tabs in den Klassen definieren, wo wir auch das Layout festgelegt haben. Allerdings können Funktionen dort nicht an die Dashboard-App eingefügt werden. Deswegen mussten wir alle Funktionen in der Dashboard-Datei erstellen.

### Beispiel für das Umsetzen eines Diagramms

Im folgenden Beispiel wird erklärt, wie die Diagramme im Dashboard angezeigt werden.

Das Layout wird zunächst in einer der Klassen für die Tabs festgelegt. Hier wird das Beispiel Angriffsvektoren von Hackerattacken dargestellt.

```

def get_layout(self):
        
    return html.Div(
        id="attackVector-wrapper",
        className="wrapper",
        children=[
            # Linke Seite des Dashboard
            html.Div(
                id="left-side",
                children=[
                    html.H3("Hackerarten und Angriffsziele"),
                    html.Pre(id='click-data'),
                    dcc.Graph(
                        id='attack-treemap', 
                        figure = self.atp.fig
                        ),
                ]),
            
            # Rechte Seite des Dashboards
            html.Div(
                id="right-side",
                children=[
                    html.H3("Informationen"),
                    html.Div(
                        id="information-wrapper", 
                        children=[
                            html.H4(id='name-attack',
                                children=[
                                    "Hinweis"
                                ]
                            ),
                            html.Div(id='info-attack',
                                children=[
                                    html.P("Klicken Sie links auf die Bereiche zu denen Sie genauere Informationen haben möchten.")
                                ]
                            ),
                        ]
                    ),
                ]
            ),

            # Footer mit Downloadbaren Inhalt
            html.Div(
                className="download-wrapper",
                children=
                [
                    html.Button("Daten herunterladen", className="btn_csv", id="attack_vectors_btn"),
                    dcc.Download(id="download-attack_vectors-excel"),
                ]
            )         
        ]
    )
```

Im Bereich **left-side** wird der Platzhalter eingefügt, in dem später das Diagramm angezeigt werden soll. Das Diagramm wird durch die Funktion in der dashboard.py Datei erzeugt.

```
atp = Chart_AttackVectors


@app.callback(
    Output('attack-treemap', 'figure'),
    Input('attack-treemap', 'clickData'))
def display_click_data(clickData):
    return atp.create_treemap()
```

Im Bereich **right-side** wird dann der Text, für den jeweils angeklickten Angriffsvektor angezeigt.

```
dbav = AttackVectorsPage(app) 

@app.callback(
    Output('info-attack', 'children'),
    Input('attack-treemap', 'clickData'))
def display_attackVectors(clickData):
    return html.P(dbav.get_information_attackVectors(clickData))
```


### dashboard.py / Einstiegspunkt

Der Einstiegspunkt der Applikation bietet die "dashboard.py" Datei. Hier wird die Dash-App erzeugt und mit dem grundlegendem Layout sowie allen Funktionen die im Dashboard gebraucht werden erzeugt. Im Layout legen wir zunächst nur das Aussehen des Headers fest.

```

app.layout = html.Div(style={'backroudColor': 'green'}, children=[
    html.Div([
        html.Header(children=[
            html.Div(
                id="app-header",
                children=[
                    # Festlegen des Titels

                    html.H1(
                        id="app-title",
                        children='Welcome to LaCTiS',
                        style={
                            
                        }
                    ),

                    # Abschnitt der einzlenen Tabs
                    dcc.Tabs(
                        id="tabs-container", 
                        value='tab_databreaches', 
                        parent_className='custom-tabs',

                        children=[
                            dcc.Tab(
                                className="custom-tab", 
                                label='Schaden durch Hacks', 
                                value='tab_databreaches',
                                selected_className='custom-tab--selected'
                            ),
                            dcc.Tab(
                                className="custom-tab", 
                                label='Hackermethoden', 
                                value='tab_methods',
                                selected_className='custom-tab--selected'
                            ),
                            dcc.Tab(
                                className="custom-tab", 
                                label='Phishing', 
                                value='tab_phishing',
                                selected_className='custom-tab--selected'
                            ),
                            dcc.Tab(
                                className="custom-tab", 
                                label='Passwortsicherheit', 
                                value='tab_password',
                                selected_className='custom-tab--selected'
                            ),
                        ]
                    ),  
                ]
            ),
        ]),
        html.Br(),html.Br(),html.Br(),
        
        # Hier wird der Inhalt dynamisch je nach Tabauswahl erzeugt
        html.Div(id="tabs-content")
    ]),
     
])
```

Die Tabauswahl beeinflusst dann, welches Layout bzw. welcher Inhalt angezeigt wird:

```
@app.callback(Output('tabs-content', 'children'),
              Input('tabs-container', 'value'))
def render_content(tab):
    if tab == 'tab_password':
        return pp.get_layout()
    elif tab == 'tab_methods':
        return dbav.get_layout()
    elif tab == 'tab_databreaches':
        return dbp.get_layout()
    elif tab == 'tab_phishing':
        return phishing.get_layout()
```

### data_breaches_cost.py / Schaden durch Hacks

Hier wollen wir dem Nutzer klar machen, was für Schaden Cyber Attacken anrichten können. Der Anwender kann sehen, wie sich die Summe über die einzelnen Jahre, sowie der Jahresdurchschnitt verhält. Ebenfalls werden die größten Schäden durch Datenlecks der einzelnen bzw. betroffenen Unternehmen pro Jahr in einem Bubbleplot dargestellt. Dem Nutzer wird zusätzlich ein Slider angeboten, mit dem ein Jahr ausgewählt werden kann.

**Was nicht ging**

Eigentlich wollten wir  x-Achsenbeschriftung linksbündig machen. Allerdings bietet plotly express keine Möglichkeit, die Titel in ihrer Position anzupassen. Einer der vielen Versuche, den wir unternommen haben, war eine Annotation einzufügen, den wir als Titel verwenden können. Das Problem war, dass die Annotation nur im Prozentverhältnis vom gesamten Diagramm oder der Zeichenfläche dargestellt werden kann. Wenn das Diagramm nun in einem anderen Format benötigt wird, verschiebt es diese wieder und müsste erneut angepasst werden. Genauso erging es uns mit der y-Achsenbeschriftung, die wollten wir rechtsbündig, statt zentiert darstellen. Sodass sie am Ende der y-Achse dargestellt wird. Jedoch war auch dies nicht mit plotly express möglich. Im nächsten Schritt haben wir es noch mit der Libary plotly go versucht. Doch auch hier ist es uns nicht gelungen die Achsenbeschriftung so darzustellen wie, wir wollten. Vermutlich liegt es daran, dass die Libarys, die wir verwendet haben, eher für die schnelle Erstellung von Diagrammen geeignet ist und nicht so feine Einstellungen gemacht werden können. Eventuell könnte man sich hier noch die Libary matplotlib anschauen.

#### Summierte Schäden

Es werden für die einzelnen Jahre die Schäden summiert und dargestellt. Um dem Nutzer ein Gefühl zu geben, ob es eher zunimmt oder abnimmt, haben wir ebenfalls den Durchschnitt berechnet.
Das Diagramm besteht dabei eigentlich aus folgenden vier Diagrammen: 

- **Summe der Einzelnen Jahre**
```
sum_df = pd.DataFrame(self.df.groupby(by=['year'])['records lost'].sum()/1000).reset_index()

df_fig1 = (sum_df)
        df_fig1 = df_fig1.loc[df_fig1["year"]>= 2014]
        # print(df_fig1.head())
        fig = px.line(df_fig1, x="year", y="records lost", 
                                labels={
                                "year": "",
                                "records lost": "entstandener Schaden (in Mrd. US$)",

                                },
                                

                                title='', markers=True)

        fig.update_xaxes(showgrid=False, title_font_family="Arial", title_font_color=color)
        fig.update_yaxes(showgrid=False, title_font_family="Arial", title_font_color=color)
        fig.update_layout(
            title={
                'text': "Verlauf des enstandenen Schadens durch Datenlecks",
                'y':0.87,
                'x':0.0,
                'xref': "paper",
                'xanchor': 'left',
                'yanchor': 'top'},
            plot_bgcolor = "rgba(0,0,0,0)",
            paper_bgcolor = "rgba(0,0,0,0)",
            font_color=color,

        
            xaxis= dict(
                range=[self.df['year'].max() - 7 - 0.5, self.df['year'].max() + 0.5],
                dtick= 1,
                ticks = "outside",
                tickwidth = 1,
                tickcolor = color,
                ticklen = 8,
                tickfont = dict(family = 'Arial', size = 14),
                showline = not darkmode,
                linewidth = 1,
                linecolor = color,
                
                

                
                ),
            yaxis = dict(
                range=[0, df_fig1['records lost'].max()*1.2],
                ticks = "outside",
                tickwidth = 1,
                tickcolor = color,
                ticklen = 8,
                ticklabelposition="outside",
                showline = True,
                linewidth = 1,
                linecolor = color,
                
                ),
            )
        

        fig.update_traces(
            marker = dict(
                color = '#4DDBE3',
                size = 10,
                opacity = 0.8,
            ),
            line = dict(
                color = '#4DDBE3',
                width = 2
            ),
        )

```
- **Durchschnitt über die Jahre**
```
avg_fig = px.line(avg_year, x="year", y="avg", 
                        title='Testtitle', markers=False, line_shape='spline')
avg_fig.update_traces(
    line = dict(
        smoothing=0.8,
        color = 'rgb(159, 90, 253)',
        width = 4
    ),
)
```

- **Punktmarker welches Jahr markiert ist für die Jahressumme (Scatterplot)**
- **Punktmarker welches Jahr markiert ist für den Durchschnitt (Scatterplot)**
```
avg_point = avg_year['avg'][avg_year["year"].index(year)]
        sum_for_year = sum_df[sum_df["year"] == year].iloc[0]['records lost']
        switcher = 1
        if (sum_for_year < avg_point):
            switcher = -1
        fig2 = px.scatter({"year": [year], "avg":avg_point}, x="year", y="avg")
        fig3 = px.scatter({"year": [year], "avg":sum_for_year}, x="year", y="avg")
```


Am Ende werden hier noch die Annotations hinzugefügt, um die Punkte zu betiteln:

```
fig.add_annotation(
    x=year,
    y=avg_point,
    xref="x",
    yref="y",
    ayref="y",
    yshift=-30*switcher,
    font=dict(
        size=14,
        color="rgb(159, 90, 253)"
        ),
    text=f"Durchschnitt<br> {int(avg_point.round()):,} $".replace(",", "."),
    showarrow=False,
    bgcolor=bg_color,
)
fig.add_annotation(
    x=year,
    y=sum_for_year,
    xref="x",
    yref="y",
    ayref="y",
    yshift=50*switcher,
    font=dict(
        size=14,
        color="#4DDBE3"
        ),
    text=f"Schaden im<br>Jahr {year}:<br> {int(sum_for_year.round()):,} $".replace(",", "."),
    showarrow=False,
    bgcolor=bg_color,
    
)
```

#### Vorfälle für das ausgewählte Jahr

:::{note}

Der genaue Code für die Diagrammerstellung wird nicht dargestellt, da das Prinzip bereits im Abschnitt Summierte Schäden erläutert wurde.  

:::

Um das Diagramm gut darstellen zu können, war es notwendig, das Dataframe erst einmal zu bearbeiten (siehe Datenhandling). Dadurch war es möglich, die Kreise weiter voneinander zu trennen und nur die Namen der Firmen zu nennen, die die größten Schäden erlitten.

**Was nicht ging**

Gerne hätten, wir die Schriftgröße in den einzelnen Bubbles, der Größe des Bubbles angepasst, sodass das Wort in den Bubble reinpasst. Allerdings ist das auch nicht möglich gewesen. Wir mussten uns schon mit einem Workaround (siehe Datenhandling) helfen, bei den kleinen Bubbles, keine Beschriftung einzufügen, da es sonst viele Überlappungen gegeben hätte. 
Insgesamt stellt sich beim Bubbleplot, die Frage, ob dies die richtige Darstellungsmethode ist oder, ob man es geschickter hätte lösen können. Eventuell wäre es einfacher darstellbar und besser zu lesen gewesen, wenn wir die Darstellung einer Tabelle gewählt hätten. Wir fanden jedoch, dass der Bubbleplot schön zum anschauen ist. Außerdem hatten wir das Gefühl, dass die Möglichkeit mit dem Slider die Jahre zu verstellen und dann die Bewegung im Bubbleplot zu sehen, Spaß macht. 


### data_breaches_attack_vectors.py / Hackermethoden

Um die Hackermethoden bzw. Angriffsvektoren gut darzustellen, wollten wir die Daten als Treemap darstellen. Dabei stellt die Größe der Felder dar, wie häufig das jeweilige Angriffsziel zu einem Datenleck führt. 

```
self.fig = px.treemap(
    self.df,  
    path=[px.Constant("Angriffsziele"), 'Fehler','Angriffspunkt'], 
    values='Häufigkeit von Data Breaches',
    labels= 'Angriffspunkt',
    color_continuous_scale=[[0, 'rgb(7, 37, 66)'], [1.0, 'rgb(77, 219, 227)']], 
    maxdepth = 2, 
    color='Häufigkeit von Data Breaches', 
    hover_data=['Angriffspunkt'],
)
self.fig.update_layout(clickmode='event+select') # Um es möglich zu machen, dass der Nutzer auf Felder klicken kann 
self.fig.update_layout(
    #uniformtext=dict(minsize=12, mode='show'),
    margin = dict(t=50, l=25, r=25, b=25), 
    plot_bgcolor= 'rgba(0, 0, 0, 0)',
    paper_bgcolor= 'rgba(0, 0, 0, 0)',
    showlegend = False,
    )
self.fig.update_coloraxes(showscale=False)
```

Das Besondere hierbei ist das Anzeigen der Informationen über die verschiedenen Hackermethoden. Der Nutzer hat hier die Möglichkeit, auf Felder zu klicken, über das eher nähere Informationen haben möchte. Im Dashboard werden diese Daten neben dem Diagramm angezeigt. 

**Schwierigkeit**

Eigentlich wollten wir die Informationen direkt im Diagramm anzeigen. Leider war das Problem, dass das Diagramm von plotly Express schon klickbar ist. Durch das Anklicken wird eine kleine Animation ausgeführt, die in das Feld führt. Unsere Idee war, die Information mittig im Diagramm anzuzeigen. Leider hätten wir dafür ein neues Diagramm erstellen müssen mit einer Annotation. Durch das ständige Überschreiben der Treemap wurden zum einen die Animationen unterbrochen und wirkten dadurch komisch und nicht mehr flüssig. Außerdem mussten wir das neue Diagramm so erstellen, als hätte der Nutzer auf einen Wert geklickt. Das konnten wir zwar darstellen, aber dann gab es keine Möglichkeit wieder zurück zu kommen.

### phishing.py / Phishing


#### Phishing Arten
Durch die Seite Phishing wollten wir die Nutzer vor allem noch mal sensibilisieren, auf die Arten zu achten. Hierfür haben wir drei Donut-Diagramme erstellt, die die Häufigkeit der verschiedenen Phishingarten darstellt. Um auf den ersten Blick zu erkennen, um welche Art es sich handelt haben wir neben Text auch Bilder in den Donuts verwendet.

```
def get_link_donut(self, darkmode=True, show_text=True):
    fig = px.pie({"link/no_link": ["Link", "No"], "value": [self.link, 100-self.link]}, names="link/no_link", values="value", hole=0.7, color="link/no_link", 
    color_discrete_map={"Link": 'rgb(62, 175, 182)', "No": 'rgb(57, 81, 104)'})
    text = ""
    if show_text:
        text = self.get_text_for_dounut("link")
    return self.update_donut_fig(fig, "E269", "Link aufrufen", darkmode, text)
```

Um die Diagramme gleich aussehen zu lassen, haben wir die Funktion update_donut_fig(figure, img_name, inner_text, darkmode, text_below) geschrieben. Hier wird jede figure überarbeitet und mit den nötigen Einstellungen versehen. Im nächsten Abschnitt werden Teile des Codes abgebildet:

```
def update_donut_fig(self, fig, img_name, txt, darkmode, expl_txt="" ):
    

    # hinzufügen des Bildes
    fig.add_layout_image(
        dict(
            source=img,
            xref="paper", yref="paper",
            x=0.5, y=0.6,
            sizex=0.3, sizey=0.3,
            xanchor="center", yanchor="middle"
        )
    )

    # hinzufügen des inneren Textes
    fig.add_annotation(
        text=txt, 
        xref="paper", 
        yref="paper",
        x=0.5, y=0.45,
        showarrow=False,
        font_size=18,
        font_color=color
    )
    
    # hinzufügen des Textes unterhalbes des Diagramms
    fig.add_annotation(
        text=expl_txt, 
        xref="paper", 
        yref="paper",
        x=0.5, y=0.0,
        showarrow=False,
        font_size=16,
        font_color=color
    )

    fig.update_traces(textinfo='none', sort=False)
    fig.layout.update(showlegend=False, plot_bgcolor= 'rgba(0, 0, 0, 0)',
        paper_bgcolor= 'rgba(0, 0, 0, 0)',)
    return fig
```

#### Phishing Zielgruppen

Hier wollten wir noch einmal darstellen, wer besonders anfällig ist für Phishing-Mails. Dafür haben wir ein horizontales Balkendiagramm erstellt. Horizontal, aufgrund der langen Bezeichnungen der einzelnen Balken. Dem Nutzer wird hier die Möglichkeit geboten zu wählen, ob er die Fehlerquoten der Abteilungen oder Branchen filtern möchte. Ebenfalls kann er seine Branche/ Abteilung, indem er sie in einem Drop-Down auswählt, hervorheben.

:::{note}

Der genaue Code für die Diagrammerstellung wird nicht dargestellt, da das Prinzip bereits im Abschnitt Summierte Schäden erläutert wurde.  

:::

### password.py / Passwortsicherheit

Hier sollen die Nutzer angeregt werden vielleicht eigene Passwörter in der Passwortwolke zu finden. Diese wird mit der Python-Bibliothek wordcloud erzeugt.

```
self.df.isna().sum() # Bearbeiten des Dataframes

# Erzeugen des JSONs mit Textelemente und größe in der Wordcloud 
text = {}
for index, row in self.df.iterrows():
    text[str(row["Password"])] = row["size"]

# Erstellen der Form für die Wordcloud -> Hier wird die Form eines Kreises erstellt
x, y = np.ogrid[:500, :500]
mask = (x - 250) ** 2 + (y - 250) ** 2 > 250 ** 2 
mask = 255 * mask.astype(int)

# Farben werden je nach darkmode erzeugt
if darkmode:
    color = "rgb(7, 37, 66)"
else:
    color = "rgb(255, 255, 255)"
self.word_cloud = WordCloud(collocations = False, background_color=color,width=1920, height=1080, mask=mask).generate_from_frequencies(text)
if darkmode:
    def white_color_func(word, font_size, position,orientation,random_state=None, **kwargs):
        return "hsl(0, 100%, 100%)"
    self.word_cloud.recolor(color_func = white_color_func)
else: 
    def blue_color_func(word, font_size, position,orientation,random_state=None, **kwargs):
        return "hsl(209, 81%, 14%)"
    self.word_cloud.recolor(color_func = blue_color_func)

return self.word_cloud.to_image() # Wordcloud wird als Bild zurückgegeben
``` 


Neben der Wordcloud können Nutzer ebenfalls ihr Passwort auf die Stärke testen lassen. Die Stärke des Passworts wird so berechnet, dass alle Möglichkeiten an Zeichen genommen werden und mit der Länge des Passworts potenziert werden. Es wird davon ausgegangen, dass ein Computer 1.000.000.000 Variationen austesten kann.

```
# Formel für die Berechnung

Möglichkeiten ** Passwortlänge / VersucheProSekunde

Möglichkeiten = (Zahlen ? + 10) + (Kleinbuchstaben ? + 26) + (Großbuchstaben ? + 26) + (Sonderzeichen ? + 32)
VersucheProSekunde = 1.000.000.000
```


