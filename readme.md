# Protocolo para la búsqueda de genes usando exonerate

## Instalación

Antes de instalar [exonerate](https://github.com/nathanweeks/exonerate) debemos tener instalado en nuestro sistema git, gcc & automake.

1. Agregamos una carpeta a nuestro directorio personal y al [`$PATH`](https://en.wikipedia.org/wiki/PATH_(variable)). El siguiente paso lo debemos ejecutar una sóla vez ya que estaremos modificando nuestro archivo de control `.bashrc`

```bash
mkdir -p $HOME/bin
echo "PATH=\"\$PATH:\$HOME/bin\"" >> $HOME/.bashrc
echo "export PATH" >> $HOME/.bashrc
```

Cerramos nuestra terminal y la volvemos a abrir para que los cambios queden asentados


2. Clonamos el repositorio de exonerate en nuestra máquina.

```bash
cd $HOME
git clone https://github.com/nathanweeks/exonerate.git
cd exonerate
git checkout v2.4.0
autoreconf -f -i
```

3. Configuramos nuestra instalación (seguimos dentro de la carpeta de exonerate).

```bash
./configure --prefix=$HOME
```

4. Instalamos exonerate, este paso genera mucho output, sólo hay que fijarse que no haya errores (seguimos dentro de la carpeta de exonerate).

```bash
make
make install
```

5. Probamos que haya quedado instalado adecuadamente exonerate

```bash
which exonerate
exonerate --help
```

## Uso de exonerate

Exonerate es un programa muy versátil que permite construir alineamientos entre secuencias de distintos tipos. En su modalidad `protein2genome` es muy util para detectar la presencia de genes en genomas novedosos.

El siguiente ejercicio demuestra el uso de exonerate para encontrar genes homólogos en genomas distantes.

### Genomas a emplear

- [*Pseudomonas aeruginosa* PA14](https://www.ncbi.nlm.nih.gov/nuccore/NC_008463)
  - [fasta (nt)](NC_008463.fasta)
- [*Thermoanaerobacter* sp. X514](https://www.ncbi.nlm.nih.gov/nuccore/NC_010320)
  - [fasta (nt)](NC_010320.fasta)

### Carnadas a emplear

#### Iniciador de replicación cromosomal dnaA

- [*Pseudomonas*](https://www.ncbi.nlm.nih.gov/protein/WP_003109151)
  - [fasta (nt)](NC_008463.dnaa.nt.fasta)
  - [fasta (aa)](WP_003109151.dnaa.aa.fasta)
- [*Thermoanaerobacter*](https://www.ncbi.nlm.nih.gov/protein/WP_009052052)
  - [fasta (nt)](NC_010320.dnaa.nt.fasta)
  - [fasta (aa)](WP_009052052.dnaa.aa.fasta)

#### Rhamnosil transferasas
- [*Pseudomonas*](https://www.ncbi.nlm.nih.gov/protein/?term=Pseudomonas+aeruginosa+%5Borgn%5D+AND+Rhamnosyltransferase+%5Btitle%5D+NOT+partial+%5Btitle%5D)
  - [fasta (aa)](rhamnosyltransferases.nr.fasta)

1. Las secuencias de los genes dnaA no son similares a nivel de nucleotidos entre los dos organismos

```bash
blastn -query NC_008463.dnaa.nt.fasta -subject NC_010320.dnaa.nt.fasta
```

[Resultados BLASTn](dnaA.blastn.txt)

2. Las secuencias de los genes dnaA son similares a nivel de aminoacidos entre los dos organismos, no obstante es limitada la similitud

```bash
blastp -query WP_003109151.dnaa.aa.fasta -subject WP_009052052.dnaa.aa.fasta > dnaA.blastp.txt
```

[Resultados BLASTp](dnaA.blastp.txt)

3. Si usamos la secuencia de dnaA de *Pseudomonas* sobre el genoma de *Pseudomonas aeruginosa*, podemos encontrar la posición del gen dnaA (control positivo).

```bash
exonerate --model protein2genome --query WP_003109151.dnaa.aa.fasta --target NC_008463.fasta --geneticcode 11 --maxintron 0 --showalignment no --showvulgar no --showtargetgff yes > NC_008463.WP_003109151.exonerate.gff
```

[Control positivo *Pseudomonas*](NC_008463.WP_003109151.exonerate.gff)

4. Si usamos la secuencia de dnaA de *Thermoanaerobacter* sobre el genoma de *Thermoanaerobacter*, podemos encontrar la posición del gen dnaA (control positivo).

```bash
exonerate --model protein2genome --query WP_009052052.dnaa.aa.fasta --target NC_010320.fasta --geneticcode 11 --maxintron 0 --showalignment no --showvulgar no --showtargetgff yes > NC_010320.WP_009052052.exonerate.gff
```

[Control positivo *Thermoanaerobacter*](NC_010320.WP_009052052.exonerate.gff)

5. Si usamos las secuencia de dnaA de *Pseudomonas* sobre el genoma de *Thermoanaerobacter* podemos encontrar la posición del gen dnaA (caso control)

```bash
exonerate --model protein2genome --query WP_003109151.dnaa.aa.fasta --target NC_010320.fasta --geneticcode 11 --maxintron 0 --showalignment no --showvulgar no --showtargetgff yes > NC_010320.WP_003109151.exonerate.gff
```

[Caso control *Thermoanaerobacter*](NC_010320.WP_009052052.exonerate.gff)

6. Si usamos las secuencia de dnaA de *Thermoanaerobacter* sobre el genoma de *Pseudomonas aeruginosa* podemos encontrar la posición del gen dnaA (caso control)

```bash
exonerate --model protein2genome --query WP_009052052.dnaa.aa.fasta --target NC_008463.fasta --geneticcode 11 --maxintron 0 --showalignment no --showvulgar no --showtargetgff yes > NC_008463.WP_009052052.exonerate.gff
```

[Caso control *Pseudomonas*](NC_010320.WP_009052052.exonerate.gff)

7. Si usamos las rhamnosil transferasas de *Pseudomonas* como carnadas, sobre el genoma de *Pseudomonas aeruginosa* podemos detectar si la cepa PA14 contiene genes codificantes de rhamnosil transferasas (caso de prueba)

```bash
exonerate --model protein2genome --query rhamnosyltransferases.nr.fasta --target NC_008463.fasta --geneticcode 11 --maxintron 0 --showalignment no --showvulgar no --showtargetgff yes > NC_008463.rhamnosyltransferases.exonerate.gff
```

[Caso de prueba *Pseudomonas*](NC_008463.rhamnosyltransferases.exonerate.gff)

7. Si usamos las rhamnosil transferasas de *Pseudomonas* como carnadas, sobre el genoma de *Thermoanaerobacter* podemos detectar si la cepa X514 contiene genes codificantes de rhamnosil transferasas (caso de prueba)

```bash
exonerate --model protein2genome --query rhamnosyltransferases.nr.fasta --target NC_010320.fasta --geneticcode 11 --maxintron 0 --showalignment no --showvulgar no --showtargetgff yes > NC_010320.rhamnosyltransferases.exonerate.gff
```

[Caso de prueba *Thermoanaerobacter*](NC_010320.rhamnosyltransferases.exonerate.gff)

Tras la examinación del resultado, es posible que haya un gen entre los nucleótidos 2229274 y 2229591, no obstante la similitud es notoriamente baja.

```bash
grep -w gene NC_010320.rhamnosyltransferases.exonerate.gff
```

Finalmente, al revisar el reporte del genoma de *Thermoanaerobacter* sp. X514, podemos ver que en efecto hay un [gen](https://www.ncbi.nlm.nih.gov/nuccore/NC_010320.1?report=genbank&from=2229121&to=2230518), no obstante es posible que se trate de una asociación muy remota.

Con ésto en mente, podemos descartar que la cepa X514 de *Thermoanaerobacter* sp. posea genes codificantes de rhamnosil transferasas.


## Conclusión

A través del uso de exonerate, podemos detectar potenciales regiones dentro de un genoma que codifiquen para proteinas usando carnadas evolutivamente distantes
