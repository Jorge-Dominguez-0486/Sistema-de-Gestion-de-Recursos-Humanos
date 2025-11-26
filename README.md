## Diseño de tablas

<img width="952" height="585" alt="image" src="https://github.com/user-attachments/assets/75050b1a-383c-41aa-80a7-b309e305d6e7" />

## models.py
```python
from django.db import models

# --- 1. ENTIDADES MAESTRAS (Base) ---

class Puesto(models.Model):
    """
    Modelo para la entidad 'Puesto'. Almacena la información de los cargos o puestos de trabajo.
    """
    id_puesto = models.AutoField(primary_key=True)
    nombre_puesto = models.CharField(
        max_length=100, 
        unique=True,
        verbose_name="Nombre del Puesto"
    ) # Utilizado como clave en la relación con Empleado_RRHH (cargo)
    descripcion_puesto = models.TextField(verbose_name="Descripción del Puesto")
    salario_minimo = models.DecimalField(
        max_digits=10, 
        decimal_places=2,
        verbose_name="Salario Mínimo"
    )
    salario_maximo = models.DecimalField(
        max_digits=10, 
        decimal_places=2,
        verbose_name="Salario Máximo"
    )
    # Nivel jerárquico como un valor decimal, según la imagen.
    nivel_jerarquico = models.DecimalField(
        max_digits=10, 
        decimal_places=2,
        verbose_name="Nivel Jerárquico"
    )
    habilidades_requeridas = models.TextField(verbose_name="Habilidades Requeridas")

    class Meta:
        verbose_name = "Puesto"
        verbose_name_plural = "Puestos"

    def __str__(self):
        return self.nombre_puesto


class Departamento(models.Model):
    """
    Modelo para la entidad 'Departamento'. Almacena la información de las áreas o departamentos.
    """
    id_departamento = models.AutoField(primary_key=True)
    nombre_departamento = models.CharField(
        max_length=100, 
        unique=True,
        verbose_name="Nombre del Departamento"
    ) # Utilizado como clave en la relación con Empleado_RRHH (departamento)
    descripcion = models.TextField(verbose_name="Descripción")
    fecha_creacion = models.DateField(verbose_name="Fecha de Creación")
    num_empleados = models.IntegerField(verbose_name="Número de Empleados")
    
    # id_gerente_departamento -> Empleado_RRHH (Muchos a Uno)
    # Se utiliza 'Empleado_RRHH' como string porque aún no está definida.
    # on_delete=models.SET_NULL: Si el gerente se va, el campo se pone a NULL.
    id_gerente_departamento = models.ForeignKey(
        'Empleado_RRHH',
        on_delete=models.SET_NULL,
        null=True,
        blank=True,
        related_name='departamentos_gestionados',
        verbose_name="Gerente del Departamento"
    )

    class Meta:
        verbose_name = "Departamento"
        verbose_name_plural = "Departamentos"

    def __str__(self):
        return self.nombre_departamento


class Empleado_RRHH(models.Model):
    """
    Modelo para la entidad 'Empleado_RRHH'. Almacena la información principal de los empleados.
    El campo id_empleado se define como CharField y Primary Key (VARCHAR(100)).
    """
    id_empleado = models.CharField(
        max_length=100, 
        primary_key=True,
        verbose_name="ID Empleado"
    )
    nombre = models.CharField(max_length=100)
    apellido = models.CharField(max_length=100)
    dni = models.CharField(max_length=100, unique=True, verbose_name="DNI/Identificación")
    fecha_nacimiento = models.DateField(verbose_name="Fecha de Nacimiento")
    genero = models.CharField(max_length=1) # CHAR(1)
    direccion = models.CharField(max_length=255)
    telefono = models.CharField(max_length=20)
    email = models.EmailField(max_length=100, unique=True) # Mejor tipo para email
    fecha_contratacion = models.DateField(verbose_name="Fecha de Contratación")
    salario = models.DecimalField(
        max_digits=10, 
        decimal_places=2
    )
    estado_empleado = models.CharField(max_length=50, verbose_name="Estado del Empleado")

    # Relación Empleado_RRHH (departamento) -> Departamento (nombre_departamento) (Muchos a Uno)
    # El FK apunta a la clase Departamento. El to_field asegura que use el nombre_departamento.
    departamento = models.ForeignKey(
        Departamento,
        on_delete=models.PROTECT, # No se puede eliminar un departamento con empleados
        related_name='empleados',
        to_field='nombre_departamento',
        verbose_name="Departamento"
    )

    # Relación Empleado_RRHH (cargo) -> Puesto (nombre_puesto) (Muchos a Uno)
    # El FK apunta a la clase Puesto. El to_field asegura que use el nombre_puesto.
    cargo = models.ForeignKey(
        Puesto,
        on_delete=models.PROTECT, # No se puede eliminar un puesto con empleados
        related_name='empleados',
        to_field='nombre_puesto',
        verbose_name="Cargo / Puesto"
    )

    class Meta:
        verbose_name = "Empleado de RRHH"
        verbose_name_plural = "Empleados de RRHH"

    def __str__(self):
        return f"{self.nombre} {self.apellido} ({self.id_empleado})"


# --- 2. ENTIDADES TRANSACCIONALES ---

class Ausencia(models.Model):
    """
    Modelo para la entidad 'Ausencia'. Registra las solicitudes y estados de ausencia.
    """
    id_ausencia = models.AutoField(primary_key=True)
    
    # id_empleado -> Empleado_RRHH (Muchos a Uno)
    # on_delete=models.CASCADE: Si se elimina el empleado, se eliminan sus ausencias.
    id_empleado = models.ForeignKey(
        Empleado_RRHH,
        on_delete=models.CASCADE,
        related_name='ausencias',
        verbose_name="Empleado"
    )
    
    tipo_ausencia = models.CharField(max_length=50, verbose_name="Tipo de Ausencia")
    fecha_inicio = models.DateField(verbose_name="Fecha de Inicio")
    fecha_fin = models.DateField(verbose_name="Fecha de Fin")
    estado_aprobacion = models.CharField(max_length=50, verbose_name="Estado de Aprobación")
    motivo_ausencia = models.TextField(verbose_name="Motivo de Ausencia")
    comentarios = models.TextField(verbose_name="Comentarios")

    class Meta:
        verbose_name = "Ausencia"
        verbose_name_plural = "Ausencias"

    def __str__(self):
        return f"Ausencia de {self.id_empleado} ({self.fecha_inicio} a {self.fecha_fin})"


class Evaluacion_Desempeno(models.Model):
    """
    Modelo para la entidad 'Evaluacion_Desempeno'. Almacena las evaluaciones de rendimiento.
    """
    id_evaluacion = models.AutoField(primary_key=True)

    # id_empleado -> Empleado_RRHH (Evaluado) (Muchos a Uno)
    id_empleado = models.ForeignKey(
        Empleado_RRHH,
        on_delete=models.CASCADE,
        related_name='evaluaciones_recibidas',
        verbose_name="Empleado Evaluado"
    )

    # id_evaluador -> Empleado_RRHH (Evaluador) (Muchos a Uno)
    # on_delete=models.SET_NULL: Si el evaluador se va, el campo se pone a NULL.
    id_evaluador = models.ForeignKey(
        Empleado_RRHH,
        on_delete=models.SET_NULL,
        null=True,
        blank=True,
        related_name='evaluaciones_realizadas',
        verbose_name="Evaluador"
    )
    
    fecha_evaluacion = models.DateField(verbose_name="Fecha de Evaluación")
    # Calificación (4,2)
    calificacion = models.DecimalField(
        max_digits=4, 
        decimal_places=2
    )
    comentarios_evaluador = models.TextField(verbose_name="Comentarios del Evaluador")
    objetivos_establecidos = models.TextField(verbose_name="Objetivos Establecidos")
    fecha_proxima_evaluacion = models.DateField(verbose_name="Fecha Próxima Evaluación")

    class Meta:
        verbose_name = "Evaluación de Desempeño"
        verbose_name_plural = "Evaluaciones de Desempeño"
        # Restricción: Un empleado no puede ser evaluado dos veces el mismo día
        unique_together = ('id_empleado', 'fecha_evaluacion') 

    def __str__(self):
        return f"Eval. {self.id_empleado} - {self.fecha_evaluacion}"


class Capacitacion(models.Model):
    """
    Modelo para la entidad 'Capacitacion'. Registra los cursos o programas de formación.
    """
    id_capacitacion = models.AutoField(primary_key=True)
    nombre_capacitacion = models.CharField(
        max_length=255, 
        verbose_name="Nombre de la Capacitación"
    )
    descripcion = models.TextField(verbose_name="Descripción")
    fecha_inicio = models.DateField(verbose_name="Fecha de Inicio")
    fecha_fin_proveedor = models.DateField(verbose_name="Fecha de Fin por Proveedor")
    costo = models.DecimalField(
        max_digits=10, 
        decimal_places=2
    )
    duracion_horas = models.DecimalField(
        max_digits=10, 
        decimal_places=2,
        verbose_name="Duración (Horas)"
    )
    es_obligatoria = models.BooleanField(verbose_name="¿Es Obligatoria?")

    class Meta:
        verbose_name = "Capacitación"
        verbose_name_plural = "Capacitaciones"

    def __str__(self):
        return self.nombre_capacitacion


class Participacion_Capacitacion(models.Model):
    """
    Modelo para la entidad 'Participacion_Capacitacion'. Es una tabla de relación
    entre Empleado_RRHH y Capacitacion, y además almacena atributos propios de la participación.
    """
    id_participacion = models.AutoField(primary_key=True)

    # id_empleado -> Empleado_RRHH (Muchos a Uno)
    id_empleado = models.ForeignKey(
        Empleado_RRHH,
        on_delete=models.CASCADE,
        related_name='participaciones_capacitacion',
        verbose_name="Empleado Participante"
    )

    # id_capacitacion -> Capacitacion (Muchos a Uno)
    id_capacitacion = models.ForeignKey(
        Capacitacion,
        on_delete=models.CASCADE,
        related_name='participaciones_empleados',
        verbose_name="Capacitación"
    )

    fecha_inscripcion = models.DateField(verbose_name="Fecha de Inscripción")
    estado_participacion = models.CharField(max_length=50, verbose_name="Estado de Participación")
    
    # Calificación (4,2). Se permite null/blank si la calificación aún no está disponible.
    calificacion_obtenida = models.DecimalField(
        max_digits=4, 
        decimal_places=2,
        null=True,
        blank=True,
        verbose_name="Calificación Obtenida"
    )
    
    # Fecha de finalización. Se permite null/blank si no ha terminado.
    fecha_finalizacion = models.DateField(
        null=True,
        blank=True,
        verbose_name="Fecha de Finalización"
    )

    class Meta:
        verbose_name = "Participación en Capacitación"
        verbose_name_plural = "Participaciones en Capacitaciones"
        # Restricción: Un empleado solo se inscribe una vez a la misma capacitación
        unique_together = ('id_empleado', 'id_capacitacion')

    def __str__(self):
        return f"Part. {self.id_empleado} en {self.id_capacitacion}"
```



