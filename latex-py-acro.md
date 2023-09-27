# Organizing and Combining LaTeX Acronyms/Glossary Entries with Python (glossaries package)
## Introduction
Problem: you want to combine two different LaTeX .tex files where the glossaries and/or acronyms are defined per the [glossaries](https://www.ctan.org/pkg/glossaries) package. You could manually do this or just add all the missing acronyms as you develop the document, but that could get tedious if they're are a lot of missing acronym/glossary entries. Instead, you could use Python to automate the task. In addition, the entries can be organized along the way.

## Set-up
Going to use acronyms in this project, but they could also be glossary entries as the glossaries package handles both nearly the same. 

First I'll set-up some example Acronym.tex files. Note how they are unsorted, something we can improve on later.


```python
acro1 = """
\\newabbreviation{abba}{ABBA}{Björn & Benny, Agnetha & Frida}
\\newabbreviation{unsc}{UNSC}{United Nations Space Command}
\\newabbreviation{odst}{ODST}{Orbital Drop Shock Trooper}
\\newabbreviation{fish}{FISH}{F' It, Stuff Happens}
\\newabbreviation{gps}{GPS}{Go Pound Sound}
\\newabbreviation{evil}{EVIL}{Every Villian is Lemon}
\\newabbreviation{otr}{OTR}{over the rainbow}
\\newabbreviation{sc}{SC}{Snack Club}
\\newabbreviation{mo}{MO}{modus operandi}
\\newabbreviation{ul}{UL}{ultralight}
\\newabbreviation{blt}{BLT}{bacon lettuce tomato}
"""

acro2 = """
\\newabbreviation{abba}{ABBA}{Björn & Benny, Agnetha & Frida}
\\newabbreviation{otr}{OTR}{Optimal Test Ruminant}
\\newabbreviation{pre}{PRE}{Prototype Ruminant Evaluation}
\\newabbreviation[longplural="Ruminants Under Test"]{rut}{RUT}{Ruminant Under Test}
\\newabbreviation{EVIL}{EVIL}{Every Villian is Lemon}
\\newabbreviation{irbh}{IRBH}{I'd Rather Be Hiking!}
\\newabbreviation{hh}{HH}{hobbit head}
\\newabbreviation{gps}{GPS}{Go Pound Sound}
\\newabbreviation{crud}{CRUD}{create, read, update, and delete}
"""

with open('Acronyms1.tex','w') as f:
    f.write(acro1)

with open('Acronyms2.tex','w') as f:
    f.write(acro2)

# Pretending we didn't just create these files.
files = ['Acronyms1.tex','Acronyms2.tex']
```

## Match Pattern
We need a pattern that captures the two different cases of acronym entries for the glossaries package. In one instance is standard/normal, where there are no optional parameters set:

> \newabbreviation{abba}{ABBA}{Björn & Benny, Agnetha & Frida}

The other has optional parameters:

> \\newabbreviation[longplural="Ruminant Under Test"]{rut}{RUT}{Ruminant Under Test}

In the pattern defined below, the optional portion is covered by (\[.*?\])?

The other three parameters are covered by the three {(.*?)}


```python
import regex as re
pattern = re.compile(r'\\newabbreviation(\[.*?\])?{(.*?)}{(.*?)}{(.*?)}\n')
```

## Run pattern against file content


```python
matches_all = set()
for fn in files:
    f = open(fn)
    f_str = f.read()
    matches_f = re.findall(pattern, f_str)
    matches_all = matches_all.union(set(matches_f))
    f.close()
matches_all
```




    {('', 'EVIL', 'EVIL', 'Every Villian is Lemon'),
     ('', 'abba', 'ABBA', 'Björn & Benny, Agnetha & Frida'),
     ('', 'blt', 'BLT', 'bacon lettuce tomato'),
     ('', 'crud', 'CRUD', 'create, read, update, and delete'),
     ('', 'evil', 'EVIL', 'Every Villian is Lemon'),
     ('', 'fish', 'FISH', "F' It, Stuff Happens"),
     ('', 'gps', 'GPS', 'Go Pound Sound'),
     ('', 'hh', 'HH', 'hobbit head'),
     ('', 'irbh', 'IRBH', "I'd Rather Be Hiking!"),
     ('', 'mo', 'MO', 'modus operandi'),
     ('', 'odst', 'ODST', 'Orbital Drop Shock Trooper'),
     ('', 'otr', 'OTR', 'Optimal Test Ruminant'),
     ('', 'otr', 'OTR', 'over the rainbow'),
     ('', 'pre', 'PRE', 'Prototype Ruminant Evaluation'),
     ('', 'sc', 'SC', 'Snack Club'),
     ('', 'ul', 'UL', 'ultralight'),
     ('', 'unsc', 'UNSC', 'United Nations Space Command'),
     ('[longplural="Ruminants Under Test"]', 'rut', 'RUT', 'Ruminant Under Test')}



## Dataframize
Because I like pandas


```python
import pandas as pd
df = pd.DataFrame(matches_all,columns=['optional','acronym id','short','long'])
df
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>optional</th>
      <th>acronym id</th>
      <th>short</th>
      <th>long</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td></td>
      <td>evil</td>
      <td>EVIL</td>
      <td>Every Villian is Lemon</td>
    </tr>
    <tr>
      <th>1</th>
      <td></td>
      <td>hh</td>
      <td>HH</td>
      <td>hobbit head</td>
    </tr>
    <tr>
      <th>2</th>
      <td></td>
      <td>EVIL</td>
      <td>EVIL</td>
      <td>Every Villian is Lemon</td>
    </tr>
    <tr>
      <th>3</th>
      <td></td>
      <td>mo</td>
      <td>MO</td>
      <td>modus operandi</td>
    </tr>
    <tr>
      <th>4</th>
      <td></td>
      <td>pre</td>
      <td>PRE</td>
      <td>Prototype Ruminant Evaluation</td>
    </tr>
    <tr>
      <th>5</th>
      <td></td>
      <td>ul</td>
      <td>UL</td>
      <td>ultralight</td>
    </tr>
    <tr>
      <th>6</th>
      <td></td>
      <td>otr</td>
      <td>OTR</td>
      <td>over the rainbow</td>
    </tr>
    <tr>
      <th>7</th>
      <td>[longplural="Ruminants Under Test"]</td>
      <td>rut</td>
      <td>RUT</td>
      <td>Ruminant Under Test</td>
    </tr>
    <tr>
      <th>8</th>
      <td></td>
      <td>otr</td>
      <td>OTR</td>
      <td>Optimal Test Ruminant</td>
    </tr>
    <tr>
      <th>9</th>
      <td></td>
      <td>crud</td>
      <td>CRUD</td>
      <td>create, read, update, and delete</td>
    </tr>
    <tr>
      <th>10</th>
      <td></td>
      <td>irbh</td>
      <td>IRBH</td>
      <td>I'd Rather Be Hiking!</td>
    </tr>
    <tr>
      <th>11</th>
      <td></td>
      <td>sc</td>
      <td>SC</td>
      <td>Snack Club</td>
    </tr>
    <tr>
      <th>12</th>
      <td></td>
      <td>blt</td>
      <td>BLT</td>
      <td>bacon lettuce tomato</td>
    </tr>
    <tr>
      <th>13</th>
      <td></td>
      <td>unsc</td>
      <td>UNSC</td>
      <td>United Nations Space Command</td>
    </tr>
    <tr>
      <th>14</th>
      <td></td>
      <td>fish</td>
      <td>FISH</td>
      <td>F' It, Stuff Happens</td>
    </tr>
    <tr>
      <th>15</th>
      <td></td>
      <td>abba</td>
      <td>ABBA</td>
      <td>Björn &amp; Benny, Agnetha &amp; Frida</td>
    </tr>
    <tr>
      <th>16</th>
      <td></td>
      <td>gps</td>
      <td>GPS</td>
      <td>Go Pound Sound</td>
    </tr>
    <tr>
      <th>17</th>
      <td></td>
      <td>odst</td>
      <td>ODST</td>
      <td>Orbital Drop Shock Trooper</td>
    </tr>
  </tbody>
</table>
</div>



## Duplicate Handling
When you combine acronym entries from different documents, you'll probably find at some point that some have the same ID or the same long form. Below I identify these and set up a flag for when we generate the final .tex file. Flagging them makes it easy to manually correct the file once its generates, which I've found was better than trying to automate a correction (e.g., adding "2" to end of duplicate entry ID).


```python
df.loc[df['acronym id'].duplicated(keep=False),'duplicate flag'] = ' %%%%% DUPLICATE'
df.loc[df['long'].duplicated(keep=False),'duplicate flag'] = ' %%%%% DUPLICATE'
df
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>optional</th>
      <th>acronym id</th>
      <th>short</th>
      <th>long</th>
      <th>duplicate flag</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td></td>
      <td>evil</td>
      <td>EVIL</td>
      <td>Every Villian is Lemon</td>
      <td>%%%%% DUPLICATE</td>
    </tr>
    <tr>
      <th>1</th>
      <td></td>
      <td>hh</td>
      <td>HH</td>
      <td>hobbit head</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2</th>
      <td></td>
      <td>EVIL</td>
      <td>EVIL</td>
      <td>Every Villian is Lemon</td>
      <td>%%%%% DUPLICATE</td>
    </tr>
    <tr>
      <th>3</th>
      <td></td>
      <td>mo</td>
      <td>MO</td>
      <td>modus operandi</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>4</th>
      <td></td>
      <td>pre</td>
      <td>PRE</td>
      <td>Prototype Ruminant Evaluation</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>5</th>
      <td></td>
      <td>ul</td>
      <td>UL</td>
      <td>ultralight</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>6</th>
      <td></td>
      <td>otr</td>
      <td>OTR</td>
      <td>over the rainbow</td>
      <td>%%%%% DUPLICATE</td>
    </tr>
    <tr>
      <th>7</th>
      <td>[longplural="Ruminants Under Test"]</td>
      <td>rut</td>
      <td>RUT</td>
      <td>Ruminant Under Test</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>8</th>
      <td></td>
      <td>otr</td>
      <td>OTR</td>
      <td>Optimal Test Ruminant</td>
      <td>%%%%% DUPLICATE</td>
    </tr>
    <tr>
      <th>9</th>
      <td></td>
      <td>crud</td>
      <td>CRUD</td>
      <td>create, read, update, and delete</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>10</th>
      <td></td>
      <td>irbh</td>
      <td>IRBH</td>
      <td>I'd Rather Be Hiking!</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>11</th>
      <td></td>
      <td>sc</td>
      <td>SC</td>
      <td>Snack Club</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>12</th>
      <td></td>
      <td>blt</td>
      <td>BLT</td>
      <td>bacon lettuce tomato</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>13</th>
      <td></td>
      <td>unsc</td>
      <td>UNSC</td>
      <td>United Nations Space Command</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>14</th>
      <td></td>
      <td>fish</td>
      <td>FISH</td>
      <td>F' It, Stuff Happens</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>15</th>
      <td></td>
      <td>abba</td>
      <td>ABBA</td>
      <td>Björn &amp; Benny, Agnetha &amp; Frida</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>16</th>
      <td></td>
      <td>gps</td>
      <td>GPS</td>
      <td>Go Pound Sound</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>17</th>
      <td></td>
      <td>odst</td>
      <td>ODST</td>
      <td>Orbital Drop Shock Trooper</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
</div>



## Create Organized Content, Write It
I take the dataframe and use it as a base for building a string which will be the contents of the final Acronyms.tex file. 

I can organize the entries while I'm at it. The first letter of the entry ID is used to to alphabetize the entries. A large comment is written to clearly indicate in the file the letter groupings.


```python
df['letter'] = df['acronym id'].str[0].str.upper()
letters = list(set(df['letter']))
letters.sort()

# for alphabetizing
df = df.sort_values(by=['letter','acronym id', 'long'])
df['entry'] = '\\newabbreviation' + df['optional'] + '{' + df['acronym id'] + '}{' + \
                df['short'] + '}{' + df['long'] + '}' + df['duplicate flag'].fillna('')

# top of the file
acronym_txt = """%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%
    % ACRONYMS
%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%
"""

# function for creating file section by letter
def to_latex_str_abc(x):
    l = list(x['letter'])[0]
    # print(l)
    abc_comment = """
%========================================================================================
%    """ + l + """
%========================================================================================
"""
    df_l = df.loc[df['letter'] == l]['entry']
    entry_txt = ''
    for entry in df_l:
        entry_txt += '\t' + entry + '\n'
    return abc_comment + entry_txt
df_by_l = list(df.groupby('letter').apply(to_latex_str_abc))
acronym_txt += ''.join(df_by_l)

# Review the result
print(acronym_txt)

# write content
with open('Acronyms.tex', 'w') as f:
    f.write(acronym_txt)
```

    %%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%
        % ACRONYMS
    %%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%
    
    %========================================================================================
    %    A
    %========================================================================================
    	\newabbreviation{abba}{ABBA}{Björn & Benny, Agnetha & Frida}
    
    %========================================================================================
    %    B
    %========================================================================================
    	\newabbreviation{blt}{BLT}{bacon lettuce tomato}
    
    %========================================================================================
    %    C
    %========================================================================================
    	\newabbreviation{crud}{CRUD}{create, read, update, and delete}
    
    %========================================================================================
    %    E
    %========================================================================================
    	\newabbreviation{EVIL}{EVIL}{Every Villian is Lemon} %%%%% DUPLICATE
    	\newabbreviation{evil}{EVIL}{Every Villian is Lemon} %%%%% DUPLICATE
    
    %========================================================================================
    %    F
    %========================================================================================
    	\newabbreviation{fish}{FISH}{F' It, Stuff Happens}
    
    %========================================================================================
    %    G
    %========================================================================================
    	\newabbreviation{gps}{GPS}{Go Pound Sound}
    
    %========================================================================================
    %    H
    %========================================================================================
    	\newabbreviation{hh}{HH}{hobbit head}
    
    %========================================================================================
    %    I
    %========================================================================================
    	\newabbreviation{irbh}{IRBH}{I'd Rather Be Hiking!}
    
    %========================================================================================
    %    M
    %========================================================================================
    	\newabbreviation{mo}{MO}{modus operandi}
    
    %========================================================================================
    %    O
    %========================================================================================
    	\newabbreviation{odst}{ODST}{Orbital Drop Shock Trooper}
    	\newabbreviation{otr}{OTR}{Optimal Test Ruminant} %%%%% DUPLICATE
    	\newabbreviation{otr}{OTR}{over the rainbow} %%%%% DUPLICATE
    
    %========================================================================================
    %    P
    %========================================================================================
    	\newabbreviation{pre}{PRE}{Prototype Ruminant Evaluation}
    
    %========================================================================================
    %    R
    %========================================================================================
    	\newabbreviation[longplural="Ruminants Under Test"]{rut}{RUT}{Ruminant Under Test}
    
    %========================================================================================
    %    S
    %========================================================================================
    	\newabbreviation{sc}{SC}{Snack Club}
    
    %========================================================================================
    %    U
    %========================================================================================
    	\newabbreviation{ul}{UL}{ultralight}
    	\newabbreviation{unsc}{UNSC}{United Nations Space Command}
    


## Conclusions
I can take this Acronyms.tex file and plop it into my Overleaf+LaTeX file and optimize it from there. This script especially becomes handy when you want to combine several different large (200+ entry) acronyms lists floating around.

This little project also highlights one of the benefits of building LaTeX documents, which is how you can automate the manipulation of plain text inputs.
