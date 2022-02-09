
## PDF
Auf der Seite des Dashboards ist es möglich einen PDF-Report zu exportieren. Dieser Report dient dazu nochmal alle Daten und deren Interpretationen in schriftlicher Form zu haben. Personen die bei der Präsentation nicht dabei waren, können hier bpsw. nochmal etwas nachlesen.

Um mit Pyhton eine PDF-Datei zu erstellen haben wir zunächst eine Word-Datei erstellt und diese in eine PDF-Datei konvertiert. Diesen Weg haben wir gewählt, da für uns die Erstellung eines PDF in Python sehr kompliziert vorkam. Nach einer kleinen Testphase verschiendener Bibliotheken schien uns vor allem PyPDF2 umständlich. Dann kamen wir auf die Idee die zunächst ein Word-Dokument zu erstellen und dieses dann zu einem PDF zu konvertieren. 
Für die Erstellung der Word-Datei haben wir die Bibliothek python-docx verwendet. Diese wird häufig für die Erstellung von DOCX-Dokumenten verwendet.

### Erste Schritte
Die ersten Schritte eine Word-Datei zu erstellen, sind uns leicht gefallen. Wir haben in Word ein Template im CI von LaCTiS und erstellt und dieses dann zur weiteren Bearbeitung zunächst geladen und dann gespeichert.

# Erstellen der Word
```
document = Document('LaCTiS_Report - Template.docx')
# Bearbeiten der Word
document.save('LaCTiS_Report.docx') 
```
# Konvertieren der Word in eine PDF
```
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
Als erstes haben wir ein Deckblatt mit Einleitung eingefügt. Für den PDF-Report haben wir anschließend Kapitel aus den Tabs des Dashboards definiert. Für das Deckblatt und die weiteren Kapitel wurden Überschriften mit verschiedenen Ebenen (level) eingefügt. Am Ende jedes Kapitels wurde ein Seitenubruch eingefügt.
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
Für den PDF-Report wurden zwei Tabellen erstellt. Eine Tabelle basiert auf dem Datensatz von Schäden durch Datenlecks. Bei dieser Tabelle wurde in der zugehörigen Klasse (Data Breaches) eine Methode geschrieben, die eine Tabelle mit Plotly Go erstellt. Die Methode sucht die größten Schäden pro Jahr heraus berechnet zudem den Mittelwert für das jeweilige Jahr. Die Werte zu den entstanden Schäden für die jeweilige Organisation wurden je nach Wert spezifisch eingefärbt. Je höher der Wert desto dunkler die bläuliche Farbe.

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
Zudem wurde noch eine weitere Tabelle zum Thema Passwortsicherheit erstellt. In diesem Thema haben wir auf unserem Dashboard den Passwort-Calculator. Den konnten wir so in unserem Report nicht umsetzen. Dafür haben wir in unserem Report erklärt, wie man die Zeit berechnet, die man benötigt, um ein Passwort zu knacken. 


### Fazit zu PDF
Wie bereits oben erwähnt haben wir uns schwer mit dem reinen erstellen einer PDF-Datei in Python. Daher haben wir diesen Weg für uns gewählt. Insgesamt ist es uns dann recht einfach gefallen 