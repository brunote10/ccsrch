# CCSRCH

[![Build Status](https://travis-ci.org/adamcaudill/ccsrch.svg?branch=master)](https://travis-ci.org/adamcaudill/ccsrch)

CCSRCH is a cross-platform tool for searching filesystems for credit card information.

### Copyright

Copyright (c) 2012-2016 Adam Caudill (adam@adamcaudill.com)

Copyright (c) 2007 Mike Beekey (zaphod2718@yahoo.com)

This application is licensed under the GNU General Public License (see below & COPYING).

This project is a fork of CCSRCH as maintained by Mike Beekey at: http://sourceforge.net/projects/ccsrch/ - it is based on the last version released (1.0.3) which was released in August 2007.

### Using CCSRCH

```
Usage: ./ccsrch <options> <start path>
  where <options> are:
    -a             Limit to ascii files.
    -b             Add the byte offset into the file of the number
    -e             Include the Modify Access and Create times in terms
                   of seconds since the epoch
    -f             Just output the filename with potential PAN data
    -i <filename>  Ignore credit card numbers in this list (test cards)
    -j             Include the Modify Access and Create times in terms
                   of normal date/time
    -o <filename>  Output the data to the file <filename> vs. standard out
    -t <1 or 2>    Check if the pattern follows either a Track 1
                   or 2 format
    -T             Check for both Track 1 and Track 2 patterns
    -c             Show a count of hits per file (only when using -o)
    -s             Show live status information (only when using -o)
    -l N           Limits the number of results from a single file before going
                   on to the next file.
    -n <list>      File extensions to exclude (i.e .dll,.exe)
    -m             Mask the PAN number.
    -w             Check for card matches wrapped across lines.
-h             Usage information
```

**Examples:**

Generic search for credit card data starting in current directory with output to screen:

`ccsrch ./`

Generic search for credit card data starting in c:\storage with output to mycard.log:

`ccsrch -o mycard.log c:\storage`

Search for credit card data and check for Track 2 data formats with output to screen:

`ccsrch -t 2 ./`

Search for credit card data and check for Track 2 data formats with output to file c.log:

`ccsrch -t 2 -o c.log ./`

Search for all track data types in ascii only files and ignore known test card numbers:

`ccsrch -T -i ignore.list -a ./`

### Output

All output is tab delimited with the following order (depending on the parameters):

* Source File
* Card Type
* Card Number
* Byte Offset
* Modify Time
* Access Time
* Create Time
* Track Pattern Match

### Assumptions

The following assumptions are made throughout the program searching for the 
card numbers:

1. Cards can be a minimum of 14 numbers and up to 16 numbers.
2. Card numbers must be contiguous. The only characters ignored when processing the files are dashes, carriage returns, new line feeds, and nulls.
3. Files are treated as raw binary objects and processed one character at a time.
4. Solo and Switch cards are not processed in the prefix search.
5. Compressed or encoded files are NOT uncompressed or decoded in this version. These files should be identified separately and the program run on the decompressed or decoded versions.

**Prefix Logic**  
The following prefixes are used to validate the potential card numbers that have passed the mod 10 (Luhn) algorithm check.

Original Sources for Credit Card Prefixes:  
http://javascript.internet.com/forms/val-credit-card.html  
http://www.erikandanna.com/Business/CreditCards/credit_card_authorization.htm

### Logic Checks

```
Card Type: MasterCard
Valid Length: 16
Valid Prefixes: 51, 52, 53, 54, 55

Card Type: VISA
Valid Length: 16
Valid Prefix: 4

Card Type: Discover
Valid Length: 16
Valid Prefix: 6011

Card Type: JCB
Valid Length: 16
Valid Prefixes: 3088, 3096, 3112, 3158, 3337, 3528, 3529

Card Type: American Express
Valid Length: 15
Valid Prefixes: 34, 37

Card Type: EnRoute
Valid Length: 15
Valid Prefixes: 2014, 2149

Card Type: JCB
Valid Length: 15
Valid Prefixes: 1800, 2131, 3528, 3529

Card Type: Diners Club, Carte Blanche
Valid Length: 14
Valid Prefixes: 36, 300, 301, 302, 303, 304, 305, 380, 381, 382, 383, 384, 385, 386, 387, 388
```

### Known Issues

Una observación / queja típica es el número de falsos positivos que aún surgen. Tendrá que revisarlos manualmente y eliminarlos. Aparecerán repetidamente ciertos patrones que coinciden con todos los criterios para las tarjetas válidas, pero son claramente falsos. Además, hay ciertos archivos del sistema que claramente no deben contener datos del titular de la tarjeta y pueden ignorarse. Es posible que haya una "lista de archivos ignorados" en una nueva versión para reducir la cantidad de cosas que deben pasar, sin embargo, esto afectará la velocidad de la herramienta.

Tenga en cuenta que dado que este programa abre cada archivo y lo procesa, obviamente el tiempo de acceso (en la época segundos) cambiará. Si va a hacer análisis forense, se supone que ya ha recopilado una imagen siguiendo prácticas forenses estándar y ya ha recopilado y conservado los tiempos de MAC, o está utilizando esta herramienta en una copia de la imagen.

Para la función de búsqueda de datos de seguimiento, la herramienta solo examina los caracteres anteriores antes del número de tarjeta de crédito válido y el delimitador o el delimitador y los caracteres (por ejemplo, la fecha de vencimiento) que siguen al número de la tarjeta de crédito.

Hemos encontrado que para algunos software de POS, se generan archivos de registro que no solo se ajustan en varias líneas, sino que también insertan representaciones hexadecimales de los valores ASCII de los datos PAN. Además, estos archivos de registro pueden contener datos de seguimiento. Recuerde que la única forma en que ccsrch encontrará los datos PAN y los datos de seguimiento es si es contiguo. En ciertos casos, puede que tenga suerte porque los archivos de registro contendrán un PAN contiguo completo y se marcarán. Le recomendamos que examine visualmente los archivos identificados para confirmación. La introducción de la lógica para capturar todas las locas representaciones de almacenamiento de PAN y el seguimiento de los datos que hemos visto convertirían a esta herramienta en una bestia.

Tenga en cuenta que ccsrch recurre a través del sistema de archivos dado un directorio de inicio e intentará abrir cualquier archivo u objeto de solo lectura de uno en uno. Dado que esto podría ser un rendimiento o una carga intensiva según la carga existente en el sistema o su configuración, le recomendamos que primero ejecute la herramienta en un subconjunto o muestra de directorios para tener una idea del impacto potencial. Negamos toda responsabilidad por cualquier impacto en el rendimiento, interrupciones o problemas que pueda causar ccsrch.

### Porting

Su herramienta ha sido compilada y ejecutada con éxito en los siguientes sistemas operativos: FreeBSD, Linux, SCO 5.0.4-5.0.7, Solaris 8, AIX 4.1.X, Windows 2000, Windows XP, and Windows 7.  

### Building

Linux/Unix/Mac OSX(Intel):  

```
$ wget -O ccsrch.tar.gz https://github.com/adamcaudill/ccsrch/tarball/master
$ tar -xvzf ccsrch.tar.gz 
$ cd adamcaudill-ccsrch-<rev>/
$ make all
```

Windows:  
Install [MinGW](http://www.mingw.org/) ([installer](http://sourceforge.net/projects/mingw/files/Installer/mingw-get-inst/))  
`mingw32-make all`

Downloads
[v1.0.8 - Win32](https://adamcaudill.com/files/ccsrch-1.0.8-win32.zip)
[v1.0.8 - OSX-Intel-64]
[v1.0.8b1 - Win32]
[v1.0.7 - Win32]
[v1.0.6 - Win32]

### License

This program is free software; you can redistribute it and/or modify it under  the terms of the GNU General Public License as published by the Free  Software Foundation; either version 2 of the License, or (at your option)  any later version.
 
This program is distributed in the hope that it will be useful, but WITHOUT ANY WARRANTY; without even the implied warranty of MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the GNU General Public License for more details.
 
You should have received a copy of the GNU General Public License along with this program; if not, write to the Free Software Foundation, Inc., 59 Temple Place - Suite 330, Boston, MA  02111-1307, USA.
