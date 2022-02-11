# Reports

:::{note}
Dieses Kapitel beschreibt die Entwicklung der Reports in PDF und PowerPoint. Zudem wird dargestellt, wie die Daten auf der Dashboard-Seite in einer Exceldatei heruntergeladen werden können.
:::

## Allgemein

Die Nutzer des Dashboards haben ebenfalls die Möglichkeit, sich zusätzlich zu unserem Dashboard weitere Informationen herunterzuladen. Zum Einen können sie sich noch einmal einen Report in Form eines PDF herunterladen. Zum anderen besteht die Möglichkeit, sich die Daten in Form einer Exceldatei herunter zu laden. Da das Ziel ist, Menschen im Punkt Cyber Security zu sensibilisieren, gibt es ebenfalls die Möglichkeit, sich eine Powerpoint heruntezuladen, um andere über das Thema zu informieren.

### Generelles Vorgehen

Um die Möglichkeit zu bieten, die Dateien zu erstellen, haben wir uns zunächst die Frage nach dem Zweck der Datei gestellt. Da unterschiedliche Bibliotheken verschiedene Vor- und Nachteile haben, konnten wir so besser einschätzen, welche wir nutzen wollen, um das Ziel möglichst schnell zu erreichen. 

Wenn wir uns für eine Technologie entschieden haben, haben wir bei Problemen dennoch geschaut, wie es bei anderen Bibliotheken gemacht wird. Das hat uns besonders beim Erstellen des PDF-Reports geholfen.

### Abbildung der Diagramme

Um downloadbare Dateien zur Verfügung zu stellen, müssen die Diagramme zunächst in einer Form vorliegen, die von allen verwendeten Bibliotheken verwendet werden können. Zunächst suchten wir nach einer eleganten Art, um die Dateien zu verwalten. 

Es gab die Möglichkeit, die erzeugten Bild-Dateien, die wir aus den Diagrammen erstellen konnten in temporären Byte Buffern zu speichern. Dies hätte es ermöglicht, die Datein nur kurzfristig im Arbeitsspeicher abzulegen und dann wieder zu löschen. Allerdings waren wir uns hier nicht sicher, ob alle Bibliotheken, die wir verwenden wollten, mit diesem Datenformat umgehen können. 

Deswegen entschieden wir uns, die Daten als png-Dateien in dem Projektordner abzulegen. Auch wenn die Bilder hier dauerhaft liegen, werden sie für jeden Download neu erstellt um immer die aktuellsten Diagramme zu beinhalten. 

Beispiel für die Erzeugung eines png Bildes:

```
pg = Phishing_Graphs() # Aufrufen der Klasse Phishing Graphs
# Erstellung des PNG eines Diagramms
pg.get_fail_bar('Abteilung', None, False).write_image("fail_bar_type_name.png", width=900, height=800, scale=1)

```

### Diagramm Anpassungen

Im Dashboard und in den Dateien waren die Diagramme auch in unterschiedlichen visuellen Umgebungen. So war der Report beispielsweise mit einem weißen Hintergrund, während das Dashboard und die Powerpoint eher einen schwarzen Hintergrund haben. Deswegen haben wir bei fast allen Diagrammen den Darkmode eingeführt:

```

# Linien-Diagramm für Kosten erstellen 
def create_lineplot(self, year, darkmode=True): 
    if darkmode:
        color = "white"
        bg_color = "rgba(7, 37, 66, 0.8)"
    else:
        color = "black"  
        bg_color = "rgba(255, 255, 255, 0.8)"

    # Code bei der das Diagramm erstellt wird

    fig.update_layout(
        xaxis= dict(
                range=[self.df['year'].max() - 7 - 0.5, self.df['year'].max() + 0.5],
                dtick= 1,
                ticks = "outside",
                tickwidth = 1,
                tickcolor = color, # hier wird die Tickfarbe je nach dem Darkmode bestimmt
                ticklen = 8,
                tickfont = dict(family = 'Arial', size = 14),
                showline = not darkmode, # Bei weißen Hintergründen wird so die Achsenlinie angezeigt
                linewidth = 1,
                linecolor = color, # hier wird die Linienfarbe je nach dem Darkmode bestimmt
        )
    )
```

## Daten in Excelform

Wenn die Nutzer nur die Daten verwenden wollen, können sie diese auf jeder Seite des Dashboards runterladen. Die Daten werden dann in einer Exceldatei bereitgestellt.

### Excel Datei mit einem Tabellenblatt

Um die Datei zu erstellen, werden zunächst die Daten aus den vorhandenen Dataframes überarbeitet. Hier werden Überschriften angepasst und die unbenutzte Spalten entfernt. 

```
def get_excel_df(self):
        res = self.wc.df.copy()
        res.drop('size', inplace=True, axis=1)
        res.drop('note', inplace=True, axis=1)
        res = res.rename(
            columns={
                'Password': 'Passwort', 
                'category': 'Kategorie', 
                'rank': 'Platz', })
        return res.to_excel
```

Am Ende wird die Funktion des Dataframes to Excel zurückgegeben, die dann an das HTML-Element im Code zurückgegeben wird und ausgeführt wird, sobald der Nutzer den Button drückt. Ebenfalls werden dann der Name der Datei und der Name des Tabellenblattes festgelegt wird.

```
@app.callback(
    Output("download-password-excel", "data"),
    Input("password_btn", "n_clicks"),
    prevent_initial_call=True,
)
def download(n_clicks):
    return dcc.send_data_frame(pp.get_excel_df(), "Passwörter.xlsx", sheet_name="Passwörter")
```
### Excel Datei mit einem Tabellenblatt

Auf manchen Dashboard-Seiten sind jedoch auch mehrere Datensätze hinterlegt. Um diese Daten trotzdem vorher alle in eine Exceldatei, mit mehreren Tabellenblättern zu schreiben, muss die Tabelle vorher mit einem Excelwriter geschrieben werden.

```
def get_excel(self):
    writer = pd.ExcelWriter('Phishing.xlsx', engine="xlsxwriter") # Name der Datei wird direkt am Anfang festgelegt
    copytoexcel = pd.DataFrame(self.pg.get_fail_df())
    copytoexcel.to_excel(writer, sheet_name="Fehlerquoten") # Ein Dataframe wird in ein Tabellenblatt der Excel Datei geschrieben
    copytoexcel = pd.DataFrame(self.pg.get_lia_df())
    copytoexcel.to_excel(writer, sheet_name="Phishing Absichten")
    writer.save()
```

Wenn der Nutzer nun die Datei anfordert wird die gespeicherte Excel Datei direkt an den Nutzer gesendet.

```
@app.callback(
    Output("download-phishing-excel", "data"),
    Input("phishing_btn", "n_clicks"),
    prevent_initial_call=True,
)
def download(n_clicks):
    phishing.get_excel()
    return dcc.send_file('Phishing.xlsx')
```

### Fazit zu Excel

Da sich Pandas Dataframes leicht in Excel Dateien umwandeln lassen, ist es sehr einfach gewesen, die Daten dem Nutzer zur Verfügung zu stellen. Es mussten lediglich die Spaltennamen angepasst werden und in manchen Fällen die Datentypen so angepasst werden, dass Excel diese interpretieren konnte.

Das Ziel, den Nutzern die Daten so zur Verfügung zu stellen, konnte sehr einfach und schnell umgesetzt werden.

## Powerpoint

Es ist ebenfalls möglich, die Diagramme und Informationen als Powerpoint zu exportieren. Das Ziel der Powerpoint ist es, dem Nutzer eine Möglichkeit zu bieten, die Diagramme besser zu interpretieren. Gegebenenfalls kann die Präsentation auch von Zuhörern genutzt werden, um andere für Themen der Cyber Sicherheit zu sensibilisieren.

Um mit Python Powerpoint Dateien zu erstellen oder zu bearbeiten, haben wir uns für die Bibliothek python-pptx verwendet. Diese Bibliothek scheint ziemlich verbreitet zu sein und wird häufig als Lösung im Internet präsentiert. 

### Erste Schritte

Die ersten Schritte, um eine Powerpoint zu erstellen und zu speichern, sind sehr einfach. Auch das Layout aus unserer Hauptpräsentation (von der Firma LaCTiS) zu übernehmen ging recht einfach, da es die Möglichkeit gibt,  Templates zu laden und zu bearbeiten.

```
pp = Presentation("Template.pptx")

# Bearbeiten der Powerpoint

pp.save("Presentation.pptx")

```

### Layout

Die erste Herausforderung, die sich uns stellte, war das Layouten der einzelnen Folien. Unsere Diagramme hatten alle unterschiedliche Formate. So waren die Diagramme der Kosten eher im Querformat und die Diagramme aus Phishing hatten eher ein Hochformat.

Nachdem wir uns überlegt hatten, wie die einzelnen Folien ungefähr aussehen sollten, ging es darum diese auch so mit Python umzusetzen. Dazu hatten wir verschiedene Möglichkeiten. Zunächst haben wir versucht, die Diagramme und Texte auf einer blanken Vorlagenfolie zu platzieren. Das stellte sich jedoch als sehr schwierig heraus, da man die Positionen auf der Folie mit Abständen zum Rand definiert. Für uns war es dann jedoch nicht möglich zu berechnen, wo der Text auf die Folie geschrieben werden sollte, denn wir hatten den Abstand zum Rand, aber nicht die Größe des Diagramms. Somit wurde es eher zu einem Trial-and-Error-Spiel die Texte richtig zu platzieren, das jedes Mal von neuem losging, wenn Texte oder Diagramme angepasst wurden. **Das war also der falsche Weg.**

Deswegen fingen wir an, mit Platzhaltern zu arbeiten. Wir haben also Masterfolien in unserem Template erstellt und die Bereiche, an denen die Bilder eingefügt werden sollten, entsprechend markiert. Später konnten wir dann in Python auf die einzelnen Platzhalter zugreifen und befüllen. 

Formatierungen von Texten und sonstige graphische Äunderungen haben wir ausschließlich in den Masterfolien von Powerpoint gemacht. Dies erschien am sinnvollsten und unkompliziertesten.

### Texte einfügen

Text konnte relativ einfach hinzugefügt werden:

```
slide = pp.slides.add_slide(self.pp.slide_layouts[2]) # Auswählen des Layouts für die Seite, die in die Präsentation eingefügt wird.

title = slide.placeholders[0]
title.text = "Was gibt es für Angriffsarten, die auf den Mensch abzielen?" # Titeltext verändern

sub_text = slide.placeholders[2]
tf = sub_text.text_frame # Textframe bekommen, in dem Text eingefügt werden kann
p = tf.add_paragraph() # Neuen Paragraphen für Text hinzufügen
p.text = 'Die größten Potentiale für Hacker sind die Komprimittierten oder Schwachen Anmeldedaten von Mitarbeitern und das Versenden von Phishing-Mails' # Paragraph mit Text befüllen
p.level = 0  

```

Mit einzelnen Paragraphen konnten einzelne Unterpunkte in das Textframe eingefügt werden. Die Formatierung haben wir wie bereits gesagt in den Masterfolien hinterlegt.

### Diagramme einfügen

Bei den Diagrammen haben wir uns wiederum etwas schwer getan. Zwar konnten wir die Diagramme einfach in die Platzhalter laden, jedoch wurden diese immer abgeschnitten, wenn sie nicht die Größe des Platzhalters hatten. Um das Problem zu lösen, mussten wir bei der Erzeugung der Bilder von den Diagrammen die Höhe und Breite festlegen. Dies konnte einfach als Variable an die Funktion write_image() übergeben werden, die die Plotly Express Figuren als Bilder abspeichert.

```
Phishing_Graphs().get_fail_bar('Branche', None, True).write_image("fail_bar_mark_pp.png", width=900, height=800)

```

Nachdem das festgelegt war, mussten wir dann darauf achten, dass die Platzhalter in der Powerpoint Masterfolien dasselbe Seitenverhältnis hatten, damit nichts abgeschnitten wurde. Ebenfalls war es wichtig zu beachten, wie groß die Höhe und Breite bei der Bilderzeugung angegeben wurde, denn wenn diese zu groß war, wurde die Beschriftung teilweise unleserlich.


### Fazit zu Powerpoint

Die Python-Bibliothek für Powerpoint funktioniert im Grunde recht gut. Einfach Präsentationen können damit relativ schnell umgesetzt werden. Wenn alles funktioniert und die Präsentation erstellt wird, ist es auch sehr befriedigend zu sehen, wie die Änderungen der Diagramme automatisch in die Powerpoint übernommen werden ohne, dass man zusätzlichen Aufwand betreiben muss.

Der Weg dahin, dass alles funktioniert, würden wir aber eher als mühselig beschreiben. Die Dokumentation der python-pptx Bibliothek ist eher dürftig und oft braucht es für Lösungen Workarounds, da Funktionen oft nicht einfach so funktionieren. Ein gutes Beispiel war hier die Erstellung der Folie über die Ziele von Phishing Mails:

```

added_img = 0
added_txt = 0

# Vorbereiten der Bilddateinamen und der Texte zu den Diagrammen
img_names = ["phishing_link_pp.png", "phishing_input_pp.png", "phishing_attach_pp.png"]
img_txt = [self.pg.get_text_for_dounut("link"), self.pg.get_text_for_dounut("input"), self.pg.get_text_for_dounut("attach")]

# itterieren über die Platzhalter der Folie, da die Indizes nicht angesteuert werden konnten
for plc in phishing_slide.placeholders:
    plc_type = str(plc.placeholder_format.type) # Öfteres aufrufen des Datentyps führt zu Fehlern. Deswegen wird es als Variable gespeichert.
    if "PICTURE" in plc_type:
        plc.insert_picture(img_names[added_img])
        added_img += 1 # Wenn ein Bild eingefügt wurde wird der Bildzähler erhöht
    if "OBJECT" in plc_type:
        tf = plc.text_frame
        p = tf.add_paragraph()
        p.text = img_txt[added_txt].replace("<br>", "") # Bezeichnungen aus dem Dashboard haben <br> Elemente, die nicht angezeigt werden sollen 
        added_txt += 1 # Wenn ein Text eingefügt wurde wird der Textzähler erhöht

```


## PDF

Auf der Seite des Dashboards ist es möglich, einen PDF-Report zu exportieren. Dieser Report dient dazu, nochmal alle Daten und deren Interpretationen in schriftlicher Form zu haben. Personen, die bei der Präsentation nicht dabei waren, können hier bpsw. nochmal etwas nachlesen.

Um mit Pyhton eine PDF-Datei zu erstellen, haben wir zunächst eine Word-Datei erstellt und diese in eine PDF-Datei konvertiert. Diesen Weg haben wir gewählt, da für uns die Erstellung eines PDF in Python sehr kompliziert vorkam. Nach einer kleinen Testphase verschiedener Bibliotheken schien uns vor allem PyPDF2 umständlich. Dann kamen wir auf die Idee, die zunächst ein Word-Dokument zu erstellen und dieses dann zu einem PDF zu konvertieren. 
Für die Erstellung der Word-Datei haben wir die Bibliothek python-docx verwendet. Diese wird häufig für die Erstellung von DOCX-Dokumenten verwendet.

### Erste Schritte

Die ersten Schritte eine Word-Datei zu erstellen, sind uns leicht gefallen. Wir haben in Word ein Template im CI von LaCTiS und erstellt und dieses dann zur weiteren Bearbeitung zunächst geladen und dann gespeichert.


```
# Erstellen der Word
document = Document('LaCTiS_Report - Template.docx')
# Bearbeiten der Word
document.save('LaCTiS_Report.docx') 
```

```
# Konvertieren der Word in eine PDF
wdFormatPDF = 17
inputFile = os.path.abspath("LaCTiS_Report.docx")
outputFile = os.path.abspath("LaCTis_Report.pdf")
word = win32com.client.Dispatch('Word.Application')
doc = word.Documents.Open(inputFile)
doc.SaveAs(outputFile, FileFormat=wdFormatPDF)
doc.Close()
word.Quit()
```

### Aufbau

#### Kapitel definieren und Überschriften einfügen

Als Erstes haben wir ein Deckblatt mit Einleitung eingefügt. Für den PDF-Report haben wir anschließend Kapitel aus den Tabs des Dashboards definiert. Für das Deckblatt und die weiteren Kapitel wurden Überschriften mit verschiedenen Ebenen (level) eingefügt. Am Ende jedes Kapitels wurde ein Seitenumbruch eingefügt.
```
#Deckblatt Überschrift
header1 = 'CYBER SECURITY REPORT '
heading = document.add_heading(header1 + '|' + actual_year, level = 0) # Erstellen Deckblatt mit dem aktuellen Jahr (das aktuelle Jahr bezieht sich immer auf das letzte vollständige vergange Jahr)

#weitere Überschriften
heading2 = document.add_heading('Schaden durch Hacks', level = 1)
#Unterüberschriften mit Paragraph-Abschnitten
heading3 = document.add_paragraph().add_run('Enstehung von Datenlecks')
heading3.bold=True
#Seitenumbruch
document.add_page_break()
```
#### Kapitel erstellen
Ein Kapitel besteht aus Diagrammen (siehe dazu Abbildung der Diagramme) und informellen Texten zu den Diagrammen. Die Diagramme wurden zentriert und eine Abbildungsbeschriftung wurde eingefügt. Die Texte wurden in Blocksatz formatiert.
Die Texte wurden als Paragraphen eingefügt. 
```
# Hinzufügen von Diagrammen
document.add_picture('fig_bubblechart.png', width=Inches(6.0))
last_paragraph = document.paragraphs[-1] 
last_paragraph.alignment = WD_ALIGN_PARAGRAPH.CENTER # Formatierung Blocksatz
abbildung_2 = document.add_paragraph('Abbildung 2: Datenlecks im Jahr 2021')
abbildung_2.paragraph_format.alignment = WD_ALIGN_PARAGRAPH.CENTER
# Text
info_db = document.add_paragraph('Daten sind ein wertvolles Gut. ...')
info_db.alignment = WD_ALIGN_PARAGRAPH.JUSTIFY # Formatierung Blocksatz
```

#### Tabellen erstellen
Für den PDF-Report wurden zwei Tabellen erstellt. Eine Tabelle basiert auf dem Datensatz von Schäden durch Datenlecks. Bei dieser Tabelle wurde in der zugehörigen Klasse (Data Breaches) eine Methode geschrieben, die eine Tabelle mit Plotly Go erstellt. Die Methode sucht die größten Schäden pro Jahr heraus und berechnet zudem den Mittelwert für das jeweilige Jahr. Die Werte zu den entstanden Schäden für die jeweilige Organisation wurden je nach Wert spezifisch eingefärbt. Je höher der Wert, desto dunkler die bläuliche Farbe. Vor allem bei dieser Tabelle haben wir uns mit der Formatierung in Word besonders schwergetan. Entweder es wurde zu groß, zu klein oder nicht zentriert. Wir haben bestimmt mehrere Stunden mit dem Trial-and-Error-Prinzip ausprobiert, die Größe optimal anzupassen, aber leider keine optimale Lösung gefunden. Das Ergebnis ist das Beste, was wir rausholen konnten. Vermutlich empfiehlt sich nächstes Mal an dieser Stelle die Erstellung der Tabelle mit python-docx. Siehe dazu Lösung mit python-docx.

```
#Tabelle für Report erstellen
    def create_table(self):
        df_fig_table = self.df.groupby('year')['records lost'].max().reset_index()


        names = []
        for index, row in df_fig_table.iterrows():
            filterd_row = self.df[(self.df['year'] == row['year']) & (self.df['records lost'] == row['records lost'])]
            names.append(filterd_row.iloc[0]['organisation'])

        df_fig_table['organisation'] = names
        df_fig_table = df_fig_table[['year', 'organisation', 'records lost']]

        df_fig_table.rename(columns={'year': 'Jahr',
                                    'organisation': 'Unternehmen',
                                    'records lost': 'Schaden in US$'}, inplace = True)
        
        df_fig_mean = self.df.groupby('year')['records lost'].mean().reset_index()
        df_fig_mean.rename(columns={'year': 'Jahr',
                                    'records lost': 'Mittelwert in US$'}, inplace = True)

        # Mittelwert Tabelle in erste Tablle einfügen
        df_fig_all = pd.merge(df_fig_table, df_fig_mean, how='inner', on='Jahr')
        
        # Schaden Zahlen in Mio. US$ angeben
        df_fig_all['Schaden in Mio. US$'] = df_fig_all['Schaden in US$'] / 1000000
        df_fig_all['Mittelwert in Mio. US$'] = df_fig_all['Mittelwert in US$'] / 1000000

        # Spalten Schäden & Mittelwert löschen
        del df_fig_all['Schaden in US$']
        del df_fig_all['Mittelwert in US$']

        #Spalte Mittelwert runden
        df_fig_all['Mittelwert in Mio. US$'] = df_fig_all['Mittelwert in Mio. US$'].round(0)

        #Zeilen löschen, nur die letzten acht Jahre anzeigen
        max_Jahre = df_fig_all['Jahr'].max()
        df_fig_all.drop(df_fig_all[df_fig_all['Jahr'] < max_Jahre - 7].index, inplace=True)

        #Farbverlauf für Schäden der Organisationen
        max_Schaden=df_fig_all['Schaden in Mio. US$'].max()
        color_blue = n_colors('rgb(7,37,66)', 'rgb(7,37,66)', 0, colortype='rgb' )
        colors = n_colors('rgb(211, 246, 248)', 'rgb(31, 188, 197)', int(max_Schaden/10), colortype='rgb')

        #Tabelle erstellen
        self.fig_table = go.Figure(data=[go.Table(
            header=dict(values=list(df_fig_all.columns),
                        fill_color='rgb(77, 219, 227)',
                        align='left',
                        font=dict(color=color_blue)),
            cells=dict(values=[df_fig_all['Jahr'], df_fig_all['Unternehmen'], df_fig_all['Schaden in Mio. US$'], df_fig_all['Mittelwert in Mio. US$']],
                    fill_color=['rgb(211, 246, 248)','rgb(211, 246, 248)', [(colors)[int(x/10)-1] for x in list(df_fig_all['Schaden in Mio. US$'])], 'rgb(211, 246, 248)'],
                    align='right'))
        ])
        return self.fig_table

```
#### Lösung in pyhton-docx        
Zudem wurde noch eine weitere Tabelle zum Thema Passwortsicherheit erstellt. In diesem Thema haben wir auf unserem Dashboard den Passwort-Calculator. Den konnten wir so in unserem Report nicht umsetzen. Dafür haben wir in unserem Report erklärt, wie man die Zeit berechnet, die man benötigt, um ein Passwort zu knacken. Die Berechnung haben wir eine Formel dargestellt und zusätzlich eine Tabelle mit Beispielen abgebildet. Diese Tabelle wurde mit Python erstellt. Das wurde von uns als sehr umständlich empfunden, vor allem wenn die Tabelle formatiert werden soll. 

```
# Daten für die Tabelle
data = [
    ['0-9', '10', '6', '1.000.000', '0,001 s'],
    ['', '', '8',  '100.000.000', '0,1 s'],
    ['', '', '10', '10.000.000.000', '10 s'],
    ['A-Z, a-z, 0-9', '10', '6', '56.800.235.584', '56 s'],
    ['', '', '8',  '218.340.105.584.896', '12 h'],
    ['', '', '10', '839.299.365.868.340.224', '26 Jahre'],
    ['A-Z, a-z, 0-9, ', '10', '6', '404.567.235.136', '11 min'],
    ['()[]{}?!$%&', '', '8',  '2.992.179.271.065.856', '13 h'],
    ['=*+~,.;:<>-_', '', '10', '22.130.157.888.803.070.976', '1700 Jahre']
]
# Erstellen der Tabelle
table_psw = document.add_table(rows = 1, cols = 5)
table_psw.style='Medium Shading 2 Accent 5' # Style der Tabelle 
# Spaltenbezeichnungen einfügen
header_cells = table_psw.rows[0].cells
header_cells[0].text = 'Verwendetes\nAlphabet'
header_cells[1].text = 'Anzahl der möglichen Zeichen'
header_cells[2].text = 'Länge des Pass-\nworts'
header_cells[3].text = 'Anzahl der möglichen Kombinationen'
header_cells[4].text = 'Zeit der vollständigen Suche'
#Tabelle mit Werten befüllen
for alphabet, anzahl, laenge, kombinationen, time in data:
    row_cells = table_psw.add_row().cells
    row_cells[0].text = alphabet.strip()
    row_cells[1].text = anzahl
    row_cells[2].text = laenge
    row_cells[3].text = kombinationen
    row_cells[3].paragraphs[0].alignment = WD_ALIGN_PARAGRAPH.RIGHT # Rechtsbündige Formatierung
    row_cells[4].text = time
# Zellen verbinden
table_psw.cell(2,1).merge(table_psw.cell(3,1))
```

### Fazit zu PDF
Wie bereits oben erwähnt, haben wir uns schwer mit dem reinen Erstellen einer PDF-Datei in Python. Wir haben uns hier auch mit Kommilitonen ausgetauscht, den es genauso ging wie uns. Daher haben wir diesen Weg für uns gewählt. Insgesamt ist es uns dann recht einfach gefallen, die Datei zu erstellen. Es gab immer mal wieder kleine Hindernisse bzw. Schwierigkeiten, die durch längeres Recherchieren aber gelöst werden konnten.
Insgesamt kann eine längere Datei zu viel Code wachsen und daher schnell unübersichtlich werden. Daher ist es wichtig, sich eine Struktur zu überlegen und den Code mit Kommentaren zu versehen. 

### Mögliche Weiterentwicklung
Aktuell werden die Nummerierungen der Tabellen- und Abbildungsbeschriftungen manuell eingefügt bzw. fix reingeschrieben. Dies hätte man durch den Einsatz von Variablen bestimmt besser lösen können.





