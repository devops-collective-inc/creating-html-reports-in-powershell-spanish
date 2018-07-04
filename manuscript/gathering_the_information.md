# Recopilación de la información

Soy un gran fan de la programación modular. Gran, gran fan. Con eso en mente, tiendo a escribir funciones que recopilan la información que quiero incluir en mi informe, y normalmente haré una función por sección principal de mi informe. Verá dentro de poco cómo es eso de beneficioso. Escribiendo cada función individualmente, hago más fácil de usar esa misma información en otras tareas, y hago más fácil depurar cada una. El truco consiste en que cada salida de función sea un solo tipo de objeto que combine toda la información de esa sección para el informe. He creado cinco funciones, que he pegado en un solo archivo de script. Le mostraré cada una de esas funciones una a la vez, con un breve comentario. Aquí va la primera:

```
function Get-InfoOS {
    [CmdletBinding()]
    param(
        [Parameter(Mandatory=$True)][string]$ComputerName
    )
    $os = Get-WmiObject -class Win32_OperatingSystem -ComputerName $ComputerName
    $props = @{'OSVersion'=$os.version;
               'SPVersion'=$os.servicepackmajorversion;
               'OSBuild'=$os.buildnumber}
    New-Object -TypeName PSObject -Property $props
}
```

Esta es una función sencilla, y la principal razón por la que me molesté en hacer que sea una función - en lugar de simplemente usar Get-WmiObject directamente - es que quiero nombres de propiedad diferentes, como "OSVersion" en lugar de sólo "Version". Dicho esto, tiendo a seguir exactamente este mismo patrón de programación para todas las funciones de recuperación de información, sólo para mantenerlas consistentes.


```
function Get-InfoCompSystem {
    [CmdletBinding()]
    param(
        [Parameter(Mandatory=$True)][string]$ComputerName
    )
    $cs = Get-WmiObject -class Win32_ComputerSystem -ComputerName $ComputerName
    $props = @{'Model'=$cs.model;
               'Manufacturer'=$cs.manufacturer;
               'RAM (GB)'="{0:N2}" -f ($cs.totalphysicalmemory / 1GB);
               'Sockets'=$cs.numberofprocessors;
               'Cores'=$cs.numberoflogicalprocessors}
    New-Object -TypeName PSObject -Property $props
}
```

Muy similar a la anterior. Notará aquí que estoy usando el operador de formato -f con la propiedad RAM, de modo que obtengo un valor en gigabytes con 2 decimales. El valor nativo está en bytes, lo que no es útil para mí.

```
function Get-InfoBadService {
    [CmdletBinding()]
    param(
        [Parameter(Mandatory=$True)][string]$ComputerName
    )
    $svcs = Get-WmiObject -class Win32_Service -ComputerName $ComputerName `
           -Filter "StartMode='Auto' AND State<>'Running'"
    foreach ($svc in $svcs) {
        $props = @{'ServiceName'=$svc.name;
                   'LogonAccount'=$svc.startname;
                   'DisplayName'=$svc.displayname}
        New-Object -TypeName PSObject -Property $props
    }
}
```

Aquí, he tenido que reconocer que voy a estar recuperando más de un objeto de WMI, así que tengo que enumerar a través de ellos usando una construcción ForEach. Una vez más, estoy principalmente renombrando propiedades. Podría haber hecho eso con un comando Select-Object, pero me gusta mantener la estructura de funciones general similar a mis otras funciones. Es sólo una preferencia personal que me ayuda a incluir menos errores, ya que estoy acostumbrado a hacer las cosas de esta manera.

```
function Get-InfoProc {
    [CmdletBinding()]
    param(
        [Parameter(Mandatory=$True)][string]$ComputerName
    )
    $procs = Get-WmiObject -class Win32_Process -ComputerName $ComputerName
    foreach ($proc in $procs) { 
        $props = @{'ProcName'=$proc.name;
                   'Executable'=$proc.ExecutablePath}
        New-Object -TypeName PSObject -Property $props
    }
}
```

Muy similar a la función de servicios. Probablemente pueda empezar a notar cómo usar esta misma estructura hace que una cierta cantidad “copiar y pegar” se vuelva efectiva cuando creo una nueva función.

```
function Get-InfoNIC {
    [CmdletBinding()]
    param(
        [Parameter(Mandatory=$True)][string]$ComputerName
    )
    $nics = Get-WmiObject -class Win32_NetworkAdapter -ComputerName $ComputerName `
           -Filter "PhysicalAdapter=True"
    foreach ($nic in $nics) {      
        $props = @{'NICName'=$nic.servicename;
                   'Speed'=$nic.speed / 1MB -as [int];
                   'Manufacturer'=$nic.manufacturer;
                   'MACAddress'=$nic.macaddress}
        New-Object -TypeName PSObject -Property $props
    }
} 
```

Lo principal para notar  aquí es cómo he convertido la propiedad speed, que esta nativamente en bytes, a megabytes. Debido a que no me interesan los decimales aquí (quiero un número entero), eligo utilizar  el valor como un entero, utilizando el operador -as, que es más sencillo para mí que el operador de formato -f. ¡Además, me da la oportunidad de mostrar esta técnica!

Tenga en cuenta que, a los efectos de este libro, voy a poner estas funciones en el mismo archivo de script que el resto de mi código, lo que en realidad genera el código HTML. Normalmente no hago eso. Normalmente, las funciones de recuperación de información van a un módulo de script, y entonces escribo mi script de generación HTML para cargar ese módulo. Tener las funciones en un módulo hace que sean más fáciles de usar en otros lugares, si quiero. Estoy omitiendo el módulo esta vez sólo para mantener las cosas más simples en esta demostración. Si desea obtener más información sobre los módulos de script, debería dar una mirada a Learn PowerShell Toolmaking in a Month of Lunches o  PowerShell in Depth, los cuales están disponibles en Manning.com.
