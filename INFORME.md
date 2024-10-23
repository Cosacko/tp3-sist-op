## Integrantes:
* Agustin Cosano
* Eliseo Facchin
* Emiliano Guardabassi
* Santiago Zuk

---

# Primera parte

1. ¿Qué política de planificación utiliza 'xv6-riscv' para elegir el próximo proceso a ejecutarse?

Xv6 utiliza una política de Round Robin (RR) para elegir el próximo proceso a ejecutarse. Xv6 mantiene
una lista de procesos que están en estado RUNNABLE y de ahí selecciona el próximo a ejecutar.

2. ¿Cuáles son los estados en los que un proceso puede permanecer en xv6-riscv y qué los hace cambiar
de estado?

Los estados en los que un proceso puede estar son: UNUSED, USED, SLEEPING, RUNNABLE, RUNNING, ZOMBIE.
* __UNUSED__: El proceso no está en uso y su estructura está libre. Cuando se llama a allocproc() pasa a USED,
cuando se llama a fork() pasa a RUNNABLE.
* __USED__: El proceso está en uso, pero no necesariamente para ejecutarse. Este estado se establece cuando se 
asigna un nuevo proceso, pero aún no está en condiciones de correr.
* __SLEEPING__: El proceso está bloqueado esperando una llamada de wakeup() para pasar a RUNNABLE.
* __RUNNABLE__: El proceso está listo para ejecutarse pero está esperando ser elegido por el scheduler.
* __RUNNING__: El proceso se está ejecutando en este momento. yield() lo hace pasar a estado RUNNABLE.
sleep() lo hace pasar de RUNNING a SLEEPING.
* __ZOMBIE__: Cuando se ejecuta exit() el proceso para a este estado hasta que su padre llame a wait(), entonces
pasa a UNUSED.

3. ¿Qué es un *quantum*?¿Dónde se define en el código?¿Cuánto dura un *quantum* en xv6-riscv?

Un *quantum* es un intervalo fijo de tiempo en el que un proceso puede estar ejecutandose en el cpu antes de 
que el sistema operativo lo interrumpa para cambiar de proceso.
En xv6 está definido en el archivo kernel/start.c:69 en la función timerinit().
Un *quantum* en este caso dura 1000000 ciclos de cpu, como 1/10 partes de segundo en qemu.

4. ¿En qué parte del código ocurre el cambio de contexto en xv6-riscv? ¿En qué funciones un proceso deja de 
ser ejecutrado? ¿En qué funciones se elige el nuevo proceso a ejecutar?

El cambio de contexto ocurre en la función swtch() que guarda los registros del proceso que estaba ejecutandose
y carga los registros del proceso que va a ejecutandose.
Un proceso deja de ser ejecutado cuando se llaman las funciones yield() que cede el cpu, sleep() que se llama 
normalmente cuando hay un I/O y bloquea el programa, exit() que termina el programa y cede el cpu.

En la función scheduler() recorre una lista de procesos y el primero que encuentre en estado RUNNABLE va a ser 
ejecutando en la proxima interrupción. Una vez seleccionado el proceso le llama a swtch() que realiza el 
context switch para ejecutar el nuevo proceso.

5. ¿El cambio de contexto consume tiempo de un *quantum*?

Sí, si consume tiempo del *quantum*. Durante un cambio de contexto, el *SO* debe guardar el estado del proceso actual,
y restaurar el estado del próximo proceso que se va a ejecutar. Esto implica operaciones que, aunque sean rápidas,
requieren algo de tiempo para completarse.
