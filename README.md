## Flujo Multifásico

Para el desarrollo del caso se usa [OpenFOAM](https://www.openfoam.com/) en su versión v2012 para Windows 10, en el bash de [Ubuntu 18.04 LTS](https://www.microsoft.com/es-ad/p/ubuntu-1804-lts/9n9tngvndl3q?activetab=pivot:overviewtab). También, para la construcción de la geometría se emplea el software [Salome](https://www.salome-platform.org/user-section) en su versión 8.3 y para la visualización de los datos se emplea [ParaView](https://www.paraview.org/download/).

### Geometria 

La geometría corresponde a un canal rectangular con un resalto hidráulico hacia el centro longitudinal del mismo, esto fue generado con la herramienta Salome. 

![Medidas_Geometria](https://user-images.githubusercontent.com/71046053/140184528-aee13ded-2ea2-48b5-a67e-005c64b90601.png)

### Malla Computacional

Esta se construye inicialmente con tetraedros en la herramienta Salome y se discrimina en 4 grupos de la siguiente manera:

- inlet: Entrada del Flujo
- outlet: Salida del Flujo
- container: Paredes y lecho del canal
- atmosphere: Borde libre

Gráficamente se observan como grupos de caras así:

![Geometria_FlujoImplicit](https://user-images.githubusercontent.com/71046053/140184521-a9ded4e2-122d-4cfd-9d19-7b3cc99de9f0.png)

Para más información, observe los archivo ubicados en el directorio *constant/polymesh* en *boundary*, donde se especifican detalladamente cada uno de los bordes. Por otro lado, como se observa en el directorio *constant/trisurface* Cada uno de estos grupos es exportado como un archivo de formato STL.  

De este modo, se procede a generar una malla base con la herramienta *BlockMesh*, configurada en el diccionario *blockMeshDict*, de la cual se parte para refinar la malla computacional en el dominio de solución con la herramienta *SnappyHexMesh*, la cual, se dispone en el diccionario *snappyHexMeshDict*.

![Malla_FlujoImplicit](https://user-images.githubusercontent.com/71046053/140184525-5ac2ee1a-e207-4092-ba69-bcb0d705a890.png)

Se aclara que solo fue empleado el sub diccionario *castelleted*, presente en el diccionario *snappyHexMesh*, para el refinamiento.

## Configuración del Caso

En primera instancia, se configura el diccionario fvSchemes para una modelación Euleriana del flujo multifásico.

## Implementación

Para implementar el caso siga ordenadamente los siguientes pasos: 

1. Tener instalados y en funcionamiento los softwares: OpenFOAM (ya sean versiones para Windows o Linux) y Paraview.
2. Clonar el repositorio. 
3. Usar el comando `checkMesh`, para verificar la integridad de la malla computacional, al final del resumen impreso en pantalla, debe decir **Mesh ok**.
5. Usar el comando `setFields`, pra definir las condiciones iniciales del término alphaWater. Debe asegurarse que el diccionario 0/alphawater este "Limpio".

```

FoamFile
{
    version     2.0;
    format      ascii;
    class       volScalarField;
    location    "0";
    object      alpha.water;
}
// * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * //

dimensions      [0 0 0 0 0 0 0];


internalField   uniform 0; 

boundaryField
{
    inlet
    {
        type            fixedValue;
        value           uniform 1;
    }
    outlet
    {
        type            zeroGradient;
    }
    container
    {
        type            zeroGradient;
    }
    atmosphere
    {
        type            inletOutlet;
        inletValue      uniform 0;
        value           uniform 0;
    }
}

```

5. En este punto puede decidir si procesa o no en paralelo, ello en función de la capacidad del hardware del que disponga, para este caso se distribuye en dos núcleos físicos el procesamiento. Use el comando `DecomposePar` para dividir la malla computacional en dos componentes y destinar un núcleo para cada uno. 
6. Use el comando `mpirun -np 2 interFoam -parallel > log &`, para iniciar los cálculos en paralelo y escribir el archivo log que será su historial del proceso.
7. Si opta por no procesar en paralelo omita los pasos 5 y 6, y ahora, tras el paso 4, use el comando `interFoam > log &`. 
8. Puede ir observando los cálculos con el comando `tail -f log`.
9. Tras terminar el proceso, si gusta ver los resultados gráficamente use el comando paraFoam -touch, para escribir su archivo de caso, y carguelo con el software paraView. 

## Recomendaciones

-
-
-
-



