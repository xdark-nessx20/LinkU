






UNIMAG Match
Sistema de Recomendación y Conexión Interdisciplinaria para la Universidad del
## Magdalena




Universidad del Magdalena



## Estudiante
## Camila Keith Florez Arias



## Programa
Ingeniería en Ciencia de Datos




Espacio académico
Prueba académica - Chic@ talento 2026




Eje temático
Innovación y Tecnología



## Fecha
## Mayo 2026




## Contenido

- Resumen ejecutivo ....................................................................................... 5
- Planteamiento del problema ......................................................................... 7
- Pregunta orientadora ................................................................................... 8
- Objetivos .................................................................................................... 9
4.1. Objetivo general ......................................................................................... 9
4.2. Objetivos específicos ................................................................................... 9
- Justificación .............................................................................................. 11
- Frase síntesis del proyecto .......................................................................... 12
- Alcance inicial del proyecto ........................................................................ 13
- Marco conceptual y fundamentos técnicos ................................................... 14
8.1. Ciencia de Datos aplicada a la vida universitaria ......................................... 14
8.2. Recolección estructurada de datos ............................................................. 15
8.3. Modelado de perfiles ................................................................................ 16
8.4. Clasificación de habilidades e intereses ...................................................... 17
8.5. Matching académico ................................................................................ 18
8.6. Sistemas de recomendación ....................................................................... 19
8.7. Análisis de similitud ................................................................................. 20

8.8. Análisis de redes ...................................................................................... 21
8.9. Visualización de datos .............................................................................. 22
8.10. Ética y uso responsable de los datos.......................................................... 23
- Metodología del proyecto ........................................................................... 23
- Herramientas propuestas para el prototipo .............................................. 30
- Ruta de implementación inicial ............................................................... 32
- Funcionamiento técnico .......................................................................... 32
11.2. Sistema de recomendación ....................................................................... 33
11.3. Visualización de resultados ...................................................................... 34
11.4. Flujo general del sistema ......................................................................... 35
- Evaluación del sistema ............................................................................ 37
- Riesgos potenciales y estrategias de mitigación ......................................... 38
- Cronograma consolidado ........................................................................ 39
- Recursos requeridos ............................................................................... 40
16.1. Recursos humanos ................................................................................. 40
16.2. Recursos técnicos ................................................................................... 40
16.3. Recursos financieros estimados ............................................................... 41
16.4. Mockups y prototipos visuales .................................................................. 41
- Viabilidad .............................................................................................. 42

- Ética y protección de datos ...................................................................... 42
- Conclusión ............................................................................................. 45
- Referencias ............................................................................................ 47
















- Resumen ejecutivo
UNIMAG Match es una propuesta académica de innovación y tecnología, desarrollada
desde la Ingeniería en Ciencia de Datos, cuyo propósito es fortalecer la colaboración
interdisciplinaria en la Universidad del Magdalena. El proyecto plantea el diseño de un sistema
inteligente de emparejamiento académico capaz de conectar estudiantes, proyectos, semilleros y
facultades mediante el análisis de habilidades, intereses, necesidades y áreas de conocimiento.
La iniciativa surge a partir de una problemática concreta: aunque la Universidad cuenta
con estudiantes talentosos, semilleros activos y proyectos con alto potencial, estos recursos
suelen permanecer dispersos y, en muchos casos, no logran articularse de manera efectiva. Como
resultado, se desaprovechan oportunidades de colaboración, innovación y desarrollo académico.
En este contexto, UNIMAG Match propone transformar los datos estudiantiles en conexiones
estratégicas, facilitando la conformación de equipos interdisciplinarios y potenciando la
generación de soluciones innovadoras.
Desde una perspectiva técnica, el proyecto se fundamenta en herramientas y
metodologías propias de la Ciencia de Datos, como la recolección estructurada de información,
el modelado de perfiles, la clasificación de habilidades, el cálculo de similitudes, los sistemas de
recomendación y la visualización de redes de colaboración. Su primera versión se concibe como
un prototipo mínimo viable, desarrollado mediante formularios digitales, bases de datos,
algoritmos básicos de coincidencia y paneles interactivos de visualización.
UNIMAG Match no pretende reemplazar la interacción humana, sino facilitarla de
manera más inteligente, transparente y organizada. Su principal valor radica en visibilizar el

talento universitario, conectar conocimientos provenientes de diferentes disciplinas y promover
la creación de equipos capaces de contribuir a la investigación, el emprendimiento, la cultura, el
bienestar, la participación estudiantil y la solución de problemáticas del entorno.
Asimismo, la propuesta se articula con el espíritu de la Semana Cultural
UNIMAGDALENA 2026, cuyo lema es "Un mapa de nuestras raíces". Así como esta temática
invita a reconocer la identidad, la diversidad y los saberes que conforman la comunidad
universitaria, UNIMAG Match busca construir un mapa dinámico del talento estudiantil, donde
cada habilidad, cada idea y cada proyecto encuentre oportunidades para conectarse, crecer y
generar impacto.










- Planteamiento del problema
En la Universidad del Magdalena, la diversidad de talentos, conocimientos e iniciativas
estudiantiles constituye uno de sus mayores activos. No obstante, la articulación entre
estudiantes, proyectos y semilleros aún presenta limitaciones significativas. Con frecuencia, los
estudiantes desconocen las oportunidades disponibles para participar en iniciativas afines a sus
habilidades e intereses, mientras que semilleros y proyectos enfrentan dificultades para
identificar perfiles adecuados que fortalezcan sus equipos de trabajo.
Esta desconexión provoca una fragmentación del potencial universitario, reduciendo las
oportunidades de colaboración interdisciplinaria y limitando el desarrollo de propuestas
innovadoras. Como consecuencia, numerosas ideas no alcanzan su máximo potencial, algunos
proyectos avanzan con restricciones de talento y se desaprovechan sinergias entre programas
académicos y facultades.
El problema central radica en la ausencia de un mecanismo institucional, sistemático y
basado en datos que facilite la identificación, visibilización y conexión estratégica entre los
distintos actores del ecosistema académico universitario. En un entorno donde los retos
contemporáneos exigen soluciones integrales e interdisciplinarias, resulta fundamental contar
con herramientas que promuevan la formación de redes colaborativas y potencien el talento
estudiantil.
En respuesta a esta necesidad, UNIMAG Match propone el desarrollo de un sistema
inteligente de emparejamiento académico orientado a fortalecer la vinculación entre estudiantes,

proyectos y semilleros, impulsando así la innovación, la investigación y la construcción colectiva
de soluciones dentro de la Universidad del Magdalena.

- Pregunta orientadora
¿Cómo diseñar e implementar un sistema inteligente de recomendación académica que
conecte estudiantes, proyectos y semilleros de la Universidad del Magdalena, a partir del análisis
de habilidades, intereses y necesidades, para fortalecer la colaboración interdisciplinaria, la
participación estudiantil y la innovación universitaria?











## 4. Objetivos
4.1. Objetivo general
Diseñar un prototipo de sistema inteligente de emparejamiento académico, basado en
Ciencia de Datos, que permita conectar estudiantes, proyectos, semilleros y facultades de la
Universidad del Magdalena a partir del análisis de habilidades, intereses y necesidades, con el fin
de fortalecer la colaboración interdisciplinaria, la innovación estudiantil y la creación de
soluciones pertinentes para la comunidad universitaria y el territorio.
4.2. Objetivos específicos
- Identificar habilidades, intereses, experiencias, áreas de conocimiento y
necesidades de estudiantes, proyectos y semilleros de la Universidad del Magdalena
mediante instrumentos de recolección de datos.
- Organizar y clasificar la información recolectada en categorías que
permitan reconocer perfiles estudiantiles, capacidades disponibles, áreas de interés y
necesidades de colaboración.
- Diseñar una lógica básica de emparejamiento académico que permita
comparar las habilidades ofrecidas por los estudiantes con las necesidades de proyectos,
semilleros o equipos de trabajo.
- Proponer un sistema de recomendación que sugiera conexiones entre
estudiantes, proyectos y facultades, favoreciendo la creación de equipos
interdisciplinarios.

- Visualizar las relaciones entre talentos, programas, habilidades y
proyectos mediante un dashboard o mapa de red que facilite la comprensión de las
oportunidades de colaboración.
- Establecer principios éticos para el uso responsable de la información
estudiantil, incluyendo consentimiento informado, privacidad, minimización de datos,
transparencia y supervisión humana.
- Plantear una ruta de implementación gradual que permita iniciar
UNIMAG Match como un prototipo mínimo viable y proyectarlo hacia futuras fases de
validación institucional.












## 5. Justificación
UNIMAG Match surge como una propuesta académica, tecnológica e institucional
orientada a responder a una necesidad real dentro de la Universidad del Magdalena: la
desconexión entre estudiantes, proyectos, semilleros e iniciativas académicas. En una comunidad
universitaria diversa, existen capacidades valiosas en múltiples áreas del conocimiento; sin
embargo, estas con frecuencia permanecen dispersas y no logran vincularse oportunamente con
oportunidades que podrían potenciarlas.
Desde la perspectiva académica, el proyecto permite aplicar conceptos propios de la
Ingeniería en Ciencia de Datos a una problemática concreta del entorno universitario. La
propuesta integra procesos de recolección estructurada de información, modelado de perfiles,
clasificación de habilidades, análisis de similitud, sistemas de recomendación y visualización de
redes colaborativas. De este modo, demuestra cómo la Ciencia de Datos puede convertirse en
una herramienta de transformación institucional.
En el ámbito institucional, UNIMAG Match contribuye al fortalecimiento de la
integración, la participación, la innovación y el sentido de pertenencia dentro de la Universidad.
Asimismo, se articula con la temática de la Semana Cultural UNIMAGDALENA 2026, "Un
mapa de nuestras raíces", al proponer la construcción de un mapa dinámico del talento
estudiantil, donde cada estudiante, semillero, programa y proyecto forme parte de una red de
colaboración activa.
Desde una perspectiva social e interdisciplinaria, la iniciativa responde a la necesidad de
conformar equipos capaces de abordar problemáticas complejas desde múltiples enfoques. Los

retos relacionados con bienestar, sostenibilidad, emprendimiento, cultura, investigación e
innovación requieren la integración de saberes provenientes de diferentes disciplinas. Facilitar la
conexión entre estudiantes de distintos programas fortalece tanto la calidad de los proyectos
como la experiencia universitaria.
Finalmente, desde el punto de vista tecnológico, el proyecto resulta viable al plantearse
inicialmente como un prototipo mínimo viable. Su primera versión puede desarrollarse mediante
formularios digitales, bases de datos, algoritmos básicos de coincidencia y dashboards
interactivos, permitiendo validar su funcionalidad y proyectar futuras etapas de escalabilidad
institucional.
En consecuencia, UNIMAG Match representa una propuesta pertinente e innovadora que
utiliza herramientas de Ciencia de Datos para visibilizar el talento universitario, fortalecer la
colaboración interdisciplinaria e impulsar el desarrollo de proyectos, semilleros y
emprendimientos con alto impacto académico y social.

- Frase síntesis del proyecto
En la Universidad no falta talento; falta una forma inteligente de conectarlo. UNIMAG
Match transforma datos estudiantiles en conexiones humanas para que las ideas encuentren
equipo, los proyectos encuentren apoyo y la Universidad innove desde la colaboración.



- Alcance inicial del proyecto
UNIMAG Match se plantea inicialmente como un prototipo académico y funcional, no
como una plataforma institucional definitiva. Su propósito en esta primera etapa es demostrar
que, mediante Ciencia de Datos, es posible transformar información estudiantil organizada en
recomendaciones útiles para conectar talentos, proyectos y necesidades dentro de la Universidad.
El alcance inicial contempla la creación de un formulario de registro para estudiantes y
proyectos, una base de datos estructurada, una lógica básica de coincidencia entre habilidades y
necesidades, y una visualización que permita observar las conexiones recomendadas. Esta
primera versión puede ser implementada con herramientas accesibles como Microsoft Forms,
Google Forms, Excel, Python, Power BI, Looker Studio o Figma.
El prototipo estaría orientado a una prueba piloto con una muestra limitada de
estudiantes, semilleros o proyectos académicos. Esta fase permitiría validar la claridad del
formulario, la utilidad de las categorías, la pertinencia de las recomendaciones y la facilidad de
interpretación del dashboard.
UNIMAG Match no pretende sustituir los procesos institucionales existentes ni tomar
decisiones automáticas sobre los estudiantes. Su función inicial es apoyar el descubrimiento de
talentos y oportunidades de colaboración. Las recomendaciones generadas por el sistema deben
entenderse como sugerencias orientadoras, no como decisiones definitivas.
A largo plazo, el proyecto podrá evolucionar hacia una plataforma institucional integrada
con semilleros, facultades, grupos estudiantiles, bienestar universitario, emprendimiento e
investigación. No obstante, esta primera etapa se concentra en validar la viabilidad del modelo y
demostrar que los datos pueden fortalecer la colaboración universitaria.


- Marco conceptual y fundamentos técnicos
UNIMAG Match se fundamenta en la aplicación de la Ciencia de Datos para resolver una
necesidad concreta de la vida universitaria: conectar talentos, proyectos, semilleros e iniciativas
estudiantiles de manera organizada, transparente y estratégica. Para ello, el proyecto integra
conceptos como recolección estructurada de datos, modelado de perfiles, clasificación de
habilidades, análisis de similitud, sistemas de recomendación, matching académico, análisis de
redes y visualización de datos.

8.1. Ciencia de Datos aplicada a la vida universitaria
La Ciencia de Datos es un campo interdisciplinario que permite recolectar, organizar,
analizar e interpretar información para apoyar la toma de decisiones. En el caso de UNIMAG
Match, la Ciencia de Datos no se utiliza únicamente para producir gráficos o almacenar
información, sino para transformar datos estudiantiles en oportunidades reales de colaboración.
Dentro de la Universidad, cada estudiante puede entenderse como una fuente de
información académica y humana: tiene habilidades, intereses, experiencias, proyectos,
necesidades y áreas donde puede aportar. Cuando esa información se organiza adecuadamente,
permite identificar patrones, afinidades y posibles conexiones entre personas y proyectos.
De esta manera, la Ciencia de Datos permite identificar relaciones estratégicas entre
estudiantes, proyectos y semilleros, facilitando la conformación de equipos interdisciplinarios y
fortaleciendo la innovación dentro de la comunidad universitaria.


8.2. Recolección estructurada de datos
El primer componente técnico de UNIMAG Match es la recolección estructurada de
información. Para que el sistema pueda recomendar conexiones útiles, primero debe contar con
datos claros, organizados y comparables.
En una primera versión, la información podría recolectarse mediante un formulario
digital dirigido a estudiantes, proyectos y semilleros. Este formulario permitiría registrar datos
como:
- programa académico;
- facultad;
- habilidades principales;
- intereses académicos;
- experiencia previa;
- disponibilidad;
- proyectos en los que participa;
- proyectos en los que desea participar;
- habilidades que puede ofrecer;
- habilidades o perfiles que necesita encontrar.

La clave no es recolectar muchos datos, sino recolectar los datos correctos. Por eso,
UNIMAG Match debe aplicar el principio de minimización: pedir únicamente la información
necesaria para generar recomendaciones académicas, evitando datos sensibles o innecesarios.

8.3. Modelado de perfiles
Después de recolectar la información, el sistema debe convertirla en perfiles organizados.
Un perfil no es solo una ficha con datos personales; es una representación estructurada de lo que
un estudiante sabe hacer, lo que le interesa y lo que puede aportar.
En UNIMAG Match existirían dos tipos principales de perfiles:
Perfil estudiantil
Incluye las habilidades, intereses, programa, facultad, experiencia, disponibilidad y áreas
de colaboración de cada estudiante.
## Ejemplo:
Una estudiante de Comunicación Social puede registrar habilidades en redacción,
producción audiovisual, manejo de redes sociales y campañas digitales.
Un estudiante de Ingeniería en Ciencias de Datos puede registrar habilidades en Python,
análisis estadístico, visualización de datos y manejo de bases de datos.
Una estudiante de Psicología puede registrar intereses en bienestar, salud mental,
comportamiento estudiantil y acompañamiento psicosocial.
Perfil de proyecto o semillero

Incluye el objetivo del proyecto, área temática, habilidades requeridas, tipo de estudiantes
que necesita, fase en la que se encuentra y posibles productos esperados.
## Ejemplo:
Un proyecto sobre bienestar estudiantil puede necesitar perfiles de Psicología, Ciencias
de Datos, Comunicación Social y Diseño.
Un proyecto ambiental puede necesitar perfiles de Ingeniería Ambiental, Biología,
Ciencias de Datos, Derecho y Comunicación.
De esta manera, UNIMAG Match organiza tanto lo que los estudiantes ofrecen como lo
que los proyectos necesitan.

8.4. Clasificación de habilidades e intereses
Para que la información sea útil, las habilidades e intereses deben clasificarse en
categorías. Esto evita que el sistema tenga datos desordenados o difíciles de comparar.
UNIMAG Match podría clasificar las habilidades en categorías como:
Tecnología y datos: programación, análisis de datos, inteligencia artificial, bases de
datos, visualización
Comunicación y divulgación: redacción, redes sociales, fotografía, video, campañas.
Investigación: búsqueda bibliográfica, formulación de proyectos, análisis de
información, escritura académica.

Diseño y creatividad: diseño gráfico, identidad visual, ilustración, edición, creación de
contenido.
Emprendimiento y gestión: modelo de negocio, finanzas, liderazgo, planeación,
mercadeo.
Bienestar y sociedad: salud mental, acompañamiento, inclusión, participación, trabajo
comunitario.
Ambiente y territorio: sostenibilidad, residuos, educación ambiental, biodiversidad,
territorio.
Cultura e identidad: arte, música, danza, patrimonio, memoria, expresiones culturales.
Esta clasificación permite que el sistema compare perfiles de manera más clara. Por
ejemplo, aunque una persona escriba “sé editar videos” y otra escriba “producción audiovisual”,
ambas habilidades pueden agruparse dentro de comunicación y divulgación.

8.5. Matching académico
El matching académico es el proceso mediante el cual el sistema identifica coincidencias
entre lo que una persona ofrece y lo que un proyecto necesita. Esta es una de las partes más
importantes de UNIMAG Match.
La lógica básica sería:
Si un proyecto necesita una habilidad y un estudiante la tiene, aumenta el nivel de
coincidencia.

Por ejemplo:
Un proyecto necesita: análisis de datos, diseño gráfico y comunicación.
Un estudiante ofrece: Python, visualización de datos y estadística.
Otro estudiante ofrece: diseño, edición y manejo de Canva.
Otro estudiante ofrece: redacción, redes sociales y comunicación estratégica.
El sistema puede recomendar a esos estudiantes como posibles integrantes del equipo.
Este proceso puede representarse mediante un puntaje de coincidencia o porcentaje de
match. Entre más habilidades, intereses o áreas comunes existan entre el estudiante y el proyecto,
mayor será la recomendación.

8.6. Sistemas de recomendación
Un sistema de recomendación es una herramienta que sugiere opciones relevantes a partir
de datos. En plataformas digitales, estos sistemas se usan para recomendar productos,
contenidos, cursos, contactos, empleos u oportunidades. En UNIMAG Match, el sistema
recomendaría conexiones académicas.
La propuesta puede iniciar con un sistema de recomendación sencillo, basado en
contenido. Esto significa que el sistema compara las características de los perfiles con las
características de los proyectos.
## Ejemplo:
Si un proyecto requiere habilidades en análisis de datos y comunicación, el sistema
buscará estudiantes que hayan registrado esas capacidades.

Si una estudiante está interesada en sostenibilidad, el sistema podrá recomendarle
proyectos ambientales, semilleros relacionados o estudiantes con intereses similares.
En una etapa inicial, no es necesario desarrollar un modelo complejo de inteligencia
artificial. Lo más importante es construir una lógica clara, explicable y viable. La investigación
avanzada recomienda que el primer prototipo trabaje con perfiles bien diseñados, categorías
claras y reglas transparentes de coincidencia.

8.7. Análisis de similitud
El análisis de similitud permite medir qué tan compatibles son dos perfiles o qué tan
adecuado es un estudiante para un proyecto. Para explicar esto ante jurados, se puede usar una
idea sencilla:
Entre más se parezcan las habilidades ofrecidas por un estudiante a las habilidades
requeridas por un proyecto, mayor será la coincidencia.
Por ejemplo:
Proyecto A necesita: Python, estadística, visualización de datos y comunicación.
Estudiante 1 tiene: Python, Excel y visualización de datos.
Estudiante 2 tiene: danza, música y actuación.
Estudiante 3 tiene: comunicación, redes sociales y edición de video.
El sistema detectaría que el Estudiante 1 tiene una coincidencia técnica alta, mientras que
el Estudiante 3 puede complementar la parte comunicativa del proyecto.

Una forma técnica de hacer esto es representar los perfiles como vectores de habilidades
y calcular la similitud entre ellos. Sin embargo, para la exposición no es necesario profundizar
demasiado en fórmulas; basta con explicar que el sistema compara habilidades, intereses y
necesidades para generar una recomendación comprensible.

8.8. Análisis de redes
UNIMAG Match no solo busca recomendar personas; también busca mostrar cómo se
conectan los talentos dentro de la Universidad. Para eso se puede usar análisis de redes.
En este enfoque, cada elemento se representa como un nodo:
- estudiantes;
- programas;
- facultades;
- habilidades;
- proyectos;
- semilleros.
Las relaciones entre ellos se representan como conexiones. Por ejemplo, un estudiante se
conecta con una habilidad, una habilidad se conecta con un proyecto, y un proyecto se conecta
con una facultad.
Esto permitiría construir un mapa visual del talento universitario. Así se podría
identificar:

qué habilidades son más frecuentes,
qué áreas tienen mayor participación,
qué programas están más conectados,
qué proyectos necesitan más apoyo,
qué talentos están poco visibles,
y qué oportunidades de colaboración existen entre facultades.
Esta idea fortalece la relación del proyecto con la Semana Cultural 2026, porque mientras
el lema habla de “Un mapa de nuestras raíces”, UNIMAG Match propone un mapa vivo del
talento, los saberes y las conexiones universitarias.

8.9. Visualización de datos
La visualización de datos permite presentar la información de forma clara, comprensible
y útil para la toma de decisiones. En UNIMAG Match, la visualización sería fundamental para
que estudiantes, semilleros o coordinadores puedan entender rápidamente las oportunidades de
conexión.
El prototipo podría incluir un dashboard con elementos como:
- número de estudiantes registrados;
- programas participantes;
- habilidades más frecuentes;
- áreas de interés más comunes;

- proyectos disponibles;
- perfiles más compatibles con cada proyecto;
- mapa de conexiones entre programas, habilidades y proyectos.
Esto permitiría que el sistema no sea solo una base de datos, sino una herramienta de
análisis y decisión. La visualización convierte la información en conocimiento y ayuda a que las
conexiones no dependan únicamente de la intuición o del contacto personal.

8.10. Ética y uso responsable de los datos
La dimensión ética constituye un principio transversal de UNIMAG Match. El uso
responsable de los datos estudiantiles, la transparencia algorítmica, la minimización de
información y la supervisión humana son elementos fundamentales del sistema. Estos principios
se desarrollan con mayor profundidad en la sección "Ética y protección de datos".

- Metodología del proyecto
La población objetivo inicial estará conformada por estudiantes activos de pregrado,
líderes de semilleros de investigación, coordinadores de proyectos estudiantiles y representantes
de iniciativas interdisciplinarias de la Universidad del Magdalena.
Para la fase piloto se proyecta trabajar con una muestra aproximada de:
- 120 estudiantes pertenecientes a diferentes facultades.

- 15 semilleros de investigación.
- 10 proyectos académicos o estudiantiles.
La selección se realizará mediante un muestreo no probabilístico por conveniencia,
priorizando la participación voluntaria y la representación de múltiples áreas del conocimiento.
Esta fase permitirá validar la claridad del instrumento, la calidad de la información
recolectada, la pertinencia de las recomendaciones generadas y la utilidad general del sistema en
un entorno controlado.
La metodología se basa en cinco fases principales: identificación del problema,
recolección de datos, organización y clasificación de perfiles, diseño de la lógica de
emparejamiento académico y visualización de las conexiones recomendadas. Esta ruta permite
que el proyecto sea comprensible, viable y escalable, sin necesidad de iniciar con una plataforma
institucional compleja.
Fase 1. Identificación de necesidades de conexión
La primera fase consiste en reconocer el problema que busca resolver UNIMAG Match:
la desconexión entre talentos, proyectos y oportunidades de colaboración dentro de la
## Universidad.
Para esta fase se identifican tres actores principales:
Estudiantes: personas con habilidades, intereses, experiencias y disposición para
participar en proyectos.
Proyectos o semilleros: iniciativas que requieren perfiles complementarios para avanzar.

Facultades y programas: espacios académicos donde existen capacidades diversas que
pueden conectarse con otras áreas del conocimiento.
Esta fase permite definir qué tipo de información necesita el sistema para generar
recomendaciones útiles. No se trata de recopilar datos sin propósito, sino de identificar qué
elementos permiten conectar adecuadamente a una persona con una oportunidad académica.
Fase 2. Recolección estructurada de datos
La segunda fase consiste en diseñar un instrumento de recolección de información, como
un formulario digital, que permita construir los perfiles de estudiantes, proyectos y semilleros.
El formulario debe ser claro, breve y orientado al objetivo del sistema. En una primera
versión, puede incluir preguntas como:
Para estudiantes:
- Nombre completo.
- Programa académico.
## • Facultad.
## • Semestre.
- Habilidades principales.
- Áreas de interés.
- Experiencia previa en proyectos.
- Herramientas que sabe utilizar.

- Tipo de proyectos en los que le gustaría participar.
- Disponibilidad aproximada.
- Rol que le gustaría asumir dentro de un equipo.
Para proyectos o semilleros:
- Nombre del proyecto o semillero.
- Programa o facultad de origen.
- Problema que busca resolver.
- Objetivo principal.
- Área temática.
- Habilidades que necesita.
- Perfiles académicos requeridos.
- Fase actual del proyecto.
- Tipo de apoyo que busca.
- Producto o resultado esperado.
Esta recolección debe cumplir con principios éticos básicos: participación voluntaria,
consentimiento informado, uso académico de la información y protección de los datos
personales. UNIMAG Match no debe recolectar información sensible ni datos innecesarios para
el proceso de emparejamiento.
Fase 3. Organización y clasificación de la información

Una vez recolectados los datos, la información debe organizarse en una base de datos
estructurada. Esta base permitirá comparar estudiantes, proyectos y semilleros de manera clara.
La información puede clasificarse en categorías como:
Datos académicos: programa, facultad, semestre.
Habilidades técnicas: programación, análisis de datos, diseño, investigación,
comunicación, formulación de proyectos.
Áreas de interés: sostenibilidad, bienestar, emprendimiento, cultura, tecnología,
educación, salud, territorio.
Tipo de participación: líder, colaborador, investigador, comunicador, diseñador, analista,
desarrollador, gestor.
Necesidades del proyecto: habilidades requeridas, perfiles buscados, tiempo disponible,
etapa del proyecto.
Esta clasificación es importante porque permite convertir respuestas abiertas en datos
comparables. Por ejemplo, si una persona escribe “manejo de redes sociales” y otra escribe
“comunicación digital”, ambas respuestas pueden agruparse dentro de una misma categoría:
comunicación y divulgación.
De esta manera, el sistema reduce el desorden de la información y facilita el cálculo de
coincidencias.
Fase 4. Diseño de la lógica de emparejamiento académico
La cuarta fase consiste en diseñar la lógica que permitirá calcular el nivel de coincidencia
entre estudiantes y proyectos.

En una primera versión, UNIMAG Match puede utilizar un sistema de puntuación basado
en reglas simples. El sistema compara lo que un proyecto necesita con lo que un estudiante
ofrece. Cada coincidencia suma puntos.
Por ejemplo:
Criterio de coincidencia Puntaje
sugerido
El estudiante tiene una habilidad requerida por el proyecto +30 puntos
El estudiante pertenece a un programa recomendado para el
proyecto
+15 puntos
El estudiante comparte el área de interés del proyecto +20 puntos
El estudiante tiene experiencia previa relacionada +20 puntos
El estudiante tiene disponibilidad compatible +15 puntos

Con esta lógica, el sistema puede generar un porcentaje de compatibilidad.
Por ejemplo:
Proyecto: campaña de bienestar estudiantil basada en datos.
Necesita: Psicología, Ciencias de Datos, Comunicación, Diseño y análisis de encuestas.
Estudiante A: sabe Python, Excel, estadística y visualización de datos.
Resultado: alto match técnico.
Estudiante B: sabe redes sociales, redacción y producción audiovisual.
Resultado: alto match comunicativo.

Estudiante C: tiene experiencia en salud mental y acompañamiento psicosocial.
Resultado: alto match humano y temático.
Así, el sistema no solo recomienda una persona, sino un posible equipo interdisciplinario.
Fase 5. Visualización de conexiones
La quinta fase consiste en mostrar los resultados de manera clara y útil mediante un
dashboard o mapa de red.
La visualización puede incluir:
- número de estudiantes registrados;
- programas participantes;
- facultades representadas;
- habilidades más frecuentes;
-  áreas de interés más solicitadas;
- proyectos disponibles;
- perfiles recomendados para cada proyecto;
- conexiones entre estudiantes, habilidades y semilleros;
- mapa de colaboración entre facultades.
Esta visualización permite que UNIMAG Match no sea únicamente una base de datos,
sino una herramienta para tomar decisiones. Los estudiantes podrían descubrir proyectos, los

semilleros podrían encontrar perfiles complementarios y la Universidad podría identificar qué
talentos existen y qué conexiones hacen falta.

- Herramientas propuestas para el prototipo
UNIMAG Match puede iniciar con herramientas accesibles y de bajo costo:
Componente Herramienta posible
Recolección de datos Google Forms o Microsoft Forms
Base de datos Excel o Google Sheets
Limpieza y análisis Python o Excel
Lógica de matching Python, Excel o reglas ponderadas
Visualización Power BI, Looker Studio o Excel
Prototipo visual Figma o Canva
Presentación final PowerPoint, Canva o Google Slides

Esta estructura permite demostrar la viabilidad del proyecto sin depender de una
aplicación compleja desde el inicio.
Ejemplo práctico de aplicación
Para explicar el funcionamiento de UNIMAG Match, se puede usar el siguiente caso:
Un grupo estudiantil quiere crear un proyecto sobre salud mental universitaria. El
equipo tiene la idea, pero necesita perfiles complementarios.

El proyecto registra que necesita:
- análisis de encuestas;
- comunicación digital;
- enfoque psicológico;
- diseño de piezas visuales;
- gestión de actividades.
UNIMAG Match analiza los perfiles estudiantiles registrados y recomienda:
- una estudiante de Ingeniería en Ciencias de Datos para analizar
información;
- un estudiante de Psicología para orientar el enfoque humano;
- una estudiante de Comunicación Social para diseñar la estrategia de
divulgación;
- un estudiante de Artes o Diseño para apoyar la identidad visual;
- un estudiante de Administración para organizar recursos y cronograma.
De esta manera, una idea que antes podía quedarse aislada se convierte en un equipo
interdisciplinario con mayor capacidad de ejecución.



- Ruta de implementación inicial
La implementación inicial de UNIMAG Match puede realizarse en tres etapas:
Etapa 1. Piloto exploratorio
Se aplica el formulario a una muestra limitada de estudiantes y proyectos, por ejemplo,
una facultad, varios semilleros o participantes voluntarios de diferentes programas.
Objetivo: validar si las preguntas permiten identificar habilidades, intereses y necesidades
reales.
Etapa 2. Prototipo funcional
Se organiza la base de datos, se diseña la lógica básica de coincidencia y se generan las
primeras recomendaciones.
Objetivo: demostrar que el sistema puede sugerir conexiones útiles entre estudiantes y
proyectos.
Etapa 3. Visualización y validación
Se construye un dashboard o mapa de red para mostrar los resultados. Luego, se revisa
con estudiantes o líderes de proyectos si las recomendaciones son pertinentes.
Objetivo: validar la utilidad del sistema y ajustar categorías, preguntas o puntajes.

- Funcionamiento técnico
El sistema transforma información estudiantil en recomendaciones académicas,
facilitando conexiones estratégicas entre estudiantes, proyectos y semilleros.

En su primera versión, UNIMAG Match funcionaría a partir de cuatro componentes
principales: cálculo de coincidencias, sistema de recomendación, visualización de resultados y
flujo general del sistema.

11.1. Cálculo de coincidencias
El componente central de UNIMAG Match es el cálculo de coincidencias, también
llamado matching académico. Este proceso permite medir qué tan compatible es un estudiante
con un proyecto o qué tan complementarios pueden ser varios estudiantes entre sí.
En una primera versión, el sistema puede utilizar una lógica de puntuación basada en
reglas simples. Cada coincidencia entre una habilidad ofrecida y una necesidad del proyecto
suma puntos. También pueden sumarse puntos por intereses comunes, experiencia relacionada,
disponibilidad compatible o pertenencia a un programa académico pertinente para el proyecto.
Por ejemplo, si un proyecto necesita análisis de datos, comunicación digital y diseño
visual, el sistema buscará estudiantes que tengan esas habilidades registradas. Luego, asignará un
porcentaje de compatibilidad para mostrar qué perfiles podrían aportar mejor al equipo.
Este puntaje no representa el valor personal del estudiante, sino el nivel de relación entre
su perfil y las necesidades específicas de una iniciativa.

11.2. Sistema de recomendación
Con base en el cálculo de coincidencias, UNIMAG Match genera recomendaciones
académicas. Estas recomendaciones pueden ser de tres tipos:

Recomendación estudiante-proyecto: sugiere proyectos en los que un estudiante podría
participar según sus habilidades e intereses.
Recomendación proyecto-estudiante: sugiere estudiantes que podrían aportar a una
iniciativa según las necesidades registradas.
Recomendación de equipo interdisciplinario: sugiere un conjunto de perfiles
complementarios para desarrollar un proyecto desde diferentes áreas del conocimiento.
Por ejemplo, para un proyecto sobre bienestar estudiantil, el sistema podría recomendar
un equipo compuesto por estudiantes de Psicología, Ingeniería en Ciencias de Datos,
Comunicación Social, Diseño y Administración.

11.3. Visualización de resultados
Los resultados del sistema se presentarían mediante un dashboard o mapa de red. Esta
visualización permitiría observar de forma clara las conexiones entre estudiantes, habilidades,
programas, facultades, proyectos y semilleros.
El dashboard podría mostrar:
número de estudiantes registrados,
programas participantes,
habilidades más frecuentes,
áreas de interés más solicitadas,
proyectos disponibles,

porcentaje de coincidencia entre perfiles y proyectos,
y mapa de conexiones interdisciplinarias.
Esta visualización convierte los datos en información útil para la toma de decisiones. Así,
los estudiantes pueden encontrar oportunidades, los proyectos pueden identificar aliados y la
Universidad puede reconocer mejor el talento disponible dentro de su comunidad.

11.4. Flujo general del sistema
El funcionamiento de UNIMAG Match puede resumirse en el siguiente flujo:
Registro de información → organización de perfiles → clasificación de habilidades
→ cálculo de coincidencias → recomendación académica → visualización de conexiones.
Impacto esperado
UNIMAG Match aportaría beneficios significativos en los ámbitos académico,
institucional y social de la comunidad universitaria.
En primer lugar, el proyecto puede beneficiar a los estudiantes, porque les permitiría
encontrar espacios donde aportar sus habilidades, participar en iniciativas de otros programas y
descubrir proyectos relacionados con sus intereses. Esto contribuye a que el estudiante no sea
visto únicamente como miembro de un programa académico, sino como parte de una red
universitaria de saberes, talentos y capacidades.
En segundo lugar, UNIMAG Match puede fortalecer los semilleros de investigación.
Muchos semilleros necesitan estudiantes con habilidades complementarias, como análisis de
datos, comunicación, diseño, formulación de proyectos, trabajo de campo o desarrollo

tecnológico. A través del sistema, estos semilleros podrían encontrar perfiles que no
necesariamente pertenecen a su mismo programa, pero que pueden aportar valor a sus líneas de
trabajo.
En tercer lugar, el proyecto puede impulsar la creación de equipos interdisciplinarios. Los
problemas reales de la Universidad y del territorio no suelen resolverse desde una sola disciplina.
Un proyecto sobre sostenibilidad puede necesitar ingeniería, comunicación, derecho, economía y
educación ambiental. Una iniciativa de bienestar puede requerir psicología, datos, comunicación
y diseño. Un emprendimiento puede necesitar administración, tecnología, mercadeo y
creatividad. UNIMAG Match facilitaría ese encuentro entre áreas.
En cuarto lugar, el sistema puede aportar al bienestar y sentido de pertenencia estudiantil.
Al hacer más fácil que los estudiantes encuentren proyectos, aliados y oportunidades de
participación, se fortalece la vida universitaria y se reduce la sensación de aislamiento
académico.
En quinto lugar, UNIMAG Match puede beneficiar el emprendimiento universitario.
Muchos estudiantes tienen ideas de negocio o iniciativas sociales, pero necesitan complementar
sus equipos con personas que sepan de finanzas, diseño, tecnología, comunicación o
formulación. El sistema permitiría conectar esas necesidades con estudiantes que tengan las
habilidades adecuadas.
En sexto lugar, el proyecto puede aportar a la cultura y a la participación estudiantil. Al
reconocer habilidades artísticas, comunicativas, tecnológicas, sociales y académicas, UNIMAG
Match puede convertirse en una herramienta para visibilizar talentos que muchas veces no
aparecen en los espacios tradicionales. Esto dialoga con el espíritu de la Semana Cultural, que

busca exaltar el talento, la identidad, la creatividad y el sentido de pertenencia de la comunidad
universitaria.
Finalmente, el proyecto puede ofrecer a la Universidad una nueva forma de leer su propio
talento. A través de mapas, dashboards y redes de conexión, sería posible identificar qué
habilidades son más frecuentes, qué áreas necesitan más apoyo, qué programas están más
conectados y dónde existen oportunidades para crear proyectos interdisciplinarios.

- Evaluación del sistema
Para medir el desempeño del prototipo durante la fase piloto, se establecerán los
siguientes indicadores:
## Indicador Descripción Meta
inicial
Precisión de
recomendaciones
Porcentaje de recomendaciones
consideradas pertinentes por los usuarios
## ≥ 75 %
Tasa de aceptación Proporción de coincidencias sugeridas
que generan contacto o colaboración real
## ≥ 60 %
Nivel de
satisfacción
Calificación promedio de los usuarios
en escala de 1 a 5
## ≥ 4.0
## Diversidad
interdisciplinaria
Número promedio de facultades
representadas por equipo recomendado
## ≥ 3

Tiempo de
respuesta
Tiempo promedio de generación de
recomendaciones
## ≤ 2
minutos
Estos indicadores permitirán evaluar objetivamente la efectividad, utilidad y pertinencia
del sistema.

- Riesgos potenciales y estrategias de mitigación
Riesgo Tipo Estrategia de mitigación
Baja participación
estudiantil
Operativo Campañas de divulgación y
articulación con semilleros y facultades
Sesgos en las
recomendaciones
Técnico Validación periódica del algoritmo
y revisión de criterios
Uso inadecuado de
datos personales
Ético Consentimiento informado,
anonimización y control de acceso
## Información
incompleta o inconsistente
Técnico Validaciones automáticas en
formularios y limpieza de datos
Resistencia al uso de
la plataforma
Adopción Capacitaciones, demostraciones y
acompañamiento inicial
La gestión anticipada de estos riesgos fortalecerá la confiabilidad y sostenibilidad del
proyecto.



- Cronograma consolidado
## Fase M
es 1
## M
es 2
## M
es 3
## M
es 4
## M
es 5
## M
es 6
## Diseño
conceptual y
metodológico
## ●

Diseño del
instrumento de
recolección
## ● ●

## Recolección
y depuración de
datos

## ● ●

## Desarrollo
del algoritmo de
matching

## ● ●

Diseño del
dashboard y
prototipo visual

## ● ●

## Implementa
ción del piloto

## ● ●


## Evaluación
y ajustes

## ●

## Presentación
final

## ●

- Recursos requeridos
16.1. Recursos humanos
- Investigador principal y líder del proyecto.
- Desarrollador de algoritmos y análisis de datos.
- Diseñador UX/UI para prototipos visuales.
- Asesor metodológico y estadístico.

16.2. Recursos técnicos
- Equipo de cómputo de alto rendimiento.
- Entorno de desarrollo en Python.
- Herramientas de visualización como Power BI.
- Plataforma de prototipado como Figma.
- Servicios de almacenamiento en la nube.


16.3. Recursos financieros estimados
Concepto Valor estimado
Licencias y herramientas COP $300.000
Material de divulgación COP $200.000
Hosting y almacenamiento COP $250.000
Contingencias COP $250.000
Total estimado COP $1.000.000

16.4. Mockups y prototipos visuales
Para facilitar la comprensión de la experiencia de usuario y del funcionamiento general
del sistema, se desarrollarán mockups funcionales utilizando herramientas como Figma y Canva.
Los prototipos incluirán:
- Pantalla de registro de estudiantes y proyectos.
- Perfil académico inteligente.
- Panel de recomendaciones personalizadas.
- Dashboard de coincidencias.
- Mapa interactivo de conexiones interdisciplinarias.
Estos elementos permitirán validar tempranamente la usabilidad, navegabilidad y
propuesta de valor de la plataforma.


## 17. Viabilidad
UNIMAG Match es una propuesta viable porque no se plantea inicialmente como una
plataforma institucional compleja, sino como un prototipo mínimo viable que puede demostrar
su funcionamiento con herramientas accesibles, de bajo costo y de fácil implementación. Esta
decisión permite que el proyecto sea realista, escalable y coherente con los recursos disponibles
para una primera fase académica.
En términos técnicos, el prototipo puede construirse con herramientas como Google
Forms o Microsoft Forms para la recolección de datos, Excel o Google Sheets para la
organización inicial, Python para la limpieza y el análisis, Power BI o Looker Studio para la
visualización, y Figma o Canva para representar la experiencia de usuario. Estas herramientas
permiten validar la lógica del sistema antes de avanzar hacia una plataforma más robusta.
Esta estrategia de implementación gradual permite reducir riesgos, optimizar recursos y
validar cada componente antes de avanzar hacia etapas de mayor complejidad.

- Ética y protección de datos
UNIMAG Match debe desarrollarse bajo principios claros de ética y protección de datos,
debido a que trabaja con información asociada a estudiantes, proyectos y posibles intereses
académicos. Desde su diseño, el sistema debe garantizar que la información recolectada sea
utilizada de manera responsable, transparente y limitada al propósito académico del proyecto.
El primer principio es el consentimiento informado. Toda persona que participe en
UNIMAG Match debe saber qué datos está entregando, para qué serán utilizados, quién podrá

acceder a ellos y cómo se protegerá su información. La participación debe ser voluntaria y
ningún estudiante debe ser obligado a registrarse para hacer parte de una actividad académica o
estudiantil.
El segundo principio es la minimización de datos. El sistema solo debe solicitar la
información necesaria para generar recomendaciones académicas, como habilidades, intereses,
programa, facultad, disponibilidad y áreas de colaboración. No debe recolectar datos sensibles,
privados o innecesarios, como información médica, biométrica, familiar, económica o cualquier
dato que pueda afectar la intimidad de los estudiantes.
El tercer principio es la privacidad. La información personal no debe publicarse de
manera abierta ni compartirse sin autorización. Los resultados del sistema deben presentarse de
forma controlada, procurando que los datos personales estén protegidos y que las
recomendaciones se utilicen únicamente para facilitar conexiones académicas.
El cuarto principio es la transparencia algorítmica. UNIMAG Match debe explicar por
qué recomienda una conexión. Por ejemplo, si el sistema sugiere a una estudiante para un
proyecto, debe indicar que la recomendación se basa en coincidencias entre sus habilidades,
intereses y las necesidades del proyecto. Esto evita que el sistema parezca una “caja negra” y
permite que las personas comprendan la lógica de las recomendaciones.
El quinto principio es la supervisión humana. Las recomendaciones del sistema no
deben tomarse como decisiones definitivas. UNIMAG Match no decide quién es mejor, quién
vale más o quién debe pertenecer a un equipo. Solo sugiere posibles conexiones que luego deben
ser revisadas y aceptadas por las personas involucradas.

El sexto principio es la prevención de sesgos. El sistema debe evitar favorecer siempre a
los perfiles más visibles, más populares o con más experiencia previa. También debe procurar
que estudiantes de diferentes programas, semestres, facultades y áreas de conocimiento tengan
oportunidades de ser recomendados.
El séptimo principio es el uso responsable de la información. Los datos no deben
emplearse para clasificar a los estudiantes por valor, rendimiento o importancia. El propósito de
UNIMAG Match no es competir por quién tiene más talento, sino ampliar las oportunidades de
colaboración entre personas con capacidades diferentes.
Por ello, la frase ética central del proyecto es:
UNIMAG Match no usa los datos para calificar personas, sino para conectar
oportunidades.
En síntesis, la protección de datos no es un elemento secundario del proyecto, sino una
condición fundamental para que UNIMAG Match sea confiable. Una herramienta que busca
conectar talentos debe hacerlo con respeto, consentimiento, transparencia y responsabilidad. Así,
el sistema puede demostrar que la innovación tecnológica también debe ser humana, ética y
justa.






## 19. Conclusión
UNIMAG Match es una propuesta académica de innovación y tecnología que demuestra
cómo la Ingeniería en Ciencias de Datos puede responder a una necesidad real de la vida
universitaria: transformar el talento universitario en redes de colaboración interdisciplinaria. La
propuesta no se limita a plantear una plataforma digital, sino que diseña un sistema inteligente de
emparejamiento académico capaz de convertir información académica en redes de colaboración
interdisciplinaria.
A través de la recolección estructurada de datos, el modelado de perfiles, la clasificación
de habilidades, el cálculo de coincidencias y la visualización de redes, UNIMAG Match permite
identificar qué estudiantes pueden aportar a determinados proyectos y qué iniciativas necesitan
perfiles complementarios para crecer.
Su valor académico está en aplicar conceptos propios de la Ciencia de Datos a un
problema institucional concreto. El sistema propone una lógica de recomendación comprensible,
viable y explicable, donde las coincidencias no se usan para calificar personas, sino para sugerir
posibles conexiones entre habilidades, intereses y necesidades. De esta manera, la tecnología se
convierte en un puente para fortalecer la participación estudiantil, la investigación, el
emprendimiento, el bienestar, la cultura y la innovación universitaria.
UNIMAG Match también se articula con la temática de la Semana Cultural
UNIMAGDALENA 2026: “Un mapa de nuestras raíces”. Así como esta temática invita a
reconocer la identidad, los saberes y la diversidad que conforman a la comunidad universitaria,
este proyecto propone construir un mapa vivo del talento estudiantil: un mapa donde cada

estudiante, programa, semillero y proyecto pueda encontrar nuevas conexiones para generar
impacto.
En síntesis, UNIMAG Match convierte la Ciencia de Datos en una herramienta humana,
académica e institucional. Su verdadero valor radica en demostrar que, cuando el talento se
organiza y se conecta, la Universidad fortalece su capacidad de innovar, colaborar y transformar
su entorno.














## 20. Referencias
Congreso de Colombia. (2012). Ley 1581 de 2012: Por la cual se dictan disposiciones
generales para la protección de datos personales. Función Pública.
https://www.funcionpublica.gov.co/eva/gestornormativo/norma.php?i=49981
Da Silva, F. L., Slodkowski, B. K., Araújo da Silva, K. K., & Cazella, S. C. (2023). A
systematic literature review on educational recommender systems for teaching and learning:
Research trends, limitations and opportunities. Education and Information Technologies, 28(3),
3289–3328. https://doi.org/10.1007/s10639-022-11341-9
Deschênes, M. (2020). Recommender systems to support learners’ agency in a learning
context: A systematic review. International Journal of Educational Technology in Higher
Education, 17, Article 50. https://doi.org/10.1186/s41239-020-00219-w
Dirección de Bienestar Universitario, Universidad del Magdalena. (2026). Concurso:
Comparsas, Chico y Chica Talento. Semana Cultural UNIMAGDALENA 2026: Un mapa de
nuestras raíces [Documento de convocatoria].
Elsevier. (s. f.). Pure: Research information management system. Recuperado el 2 de
mayo de 2026, de https://www.elsevier.com/products/pure
Google Workspace. (s. f.). Google Forms: Online form creator. Recuperado el 2 de mayo
de 2026, de https://workspace.google.com/products/forms/
Handshake. (2025). Job matches. Recuperado el 2 de mayo de 2026, de
https://support.joinhandshake.com/hc/en-gb/articles/360046808653-Job-Matches

Jacob, W. J. (2015). Interdisciplinary trends in higher education. Palgrave
Communications, 1, Article 15001. https://doi.org/10.1057/palcomms.2015.1
Jisc. (2023). Code of practice for learning analytics. https://www.jisc.ac.uk/guides/code-
of-practice-for-learning-analytics
Microsoft. (s. f.). Power BI service basic concepts for designers. Recuperado el 2 de
mayo de 2026, de https://learn.microsoft.com/en-us/power-bi/fundamentals/service-basic-
concepts
National Academies of Sciences, Engineering, and Medicine. (2005). Facilitating
interdisciplinary research. The National Academies Press. https://doi.org/10.17226/11153
NetworkX. (s. f.). NetworkX: Network analysis in Python. Recuperado el 2 de mayo de
2026, de https://networkx.org/
Organisation for Economic Co-operation and Development. (s. f.). Future of education
and skills 2030. Recuperado el 2 de mayo de 2026, de
https://www.oecd.org/en/about/projects/future-of-education-and-skills-2030.html
Organisation for Economic Co-operation and Development. (s. f.). The OECD Learning
Compass 2030. Recuperado el 2 de mayo de 2026, de https://www.oecd.org/en/data/tools/oecd-
learning-compass-2030.html
PeopleGrove. (s. f.). Mentoring platform for alumni and students. Recuperado el 2 de
mayo de 2026, de https://www.peoplegrove.com/solutions/universities/mentorship/
Python Software Foundation. (s. f.). Python. Recuperado el 2 de mayo de 2026, de
https://www.python.org/

Roy, D., & Dutta, M. (2022). A systematic review and research perspective on
recommender systems. Journal of Big Data, 9, Article 59. https://doi.org/10.1186/s40537-022-
## 00592-5
Scikit-learn. (s. f.). sklearn.metrics.pairwise.cosine_similarity. Recuperado el 2 de mayo
de 2026, de https://scikit-
learn.org/stable/modules/generated/sklearn.metrics.pairwise.cosine_similarity.html
UNESCO. (2022). Recommendation on the ethics of artificial intelligence.
https://www.unesco.org/en/articles/recommendation-ethics-artificial-intelligence
Universidad del Magdalena. (2026). Investigación profunda para sustentar UNIMAG
Match [Documento de trabajo no publicado].
VIVO. (s. f.). About VIVO. Recuperado el 2 de mayo de 2026, de
https://vivoweb.org/about/
VIVO. (s. f.). Facilitate the discovery and connection of research and scholarship.
Recuperado el 2 de mayo de 2026, de https://www.vivoweb.org/