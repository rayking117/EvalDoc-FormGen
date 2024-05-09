# EvalDoc-FormGen
Script generador de formularios para evaluacion docente.

Name: EvalDoc-FormGen

Version: 0.1

Dev: Ing. Hector Rafael Gonzalez Vega

Date: 21/11/2023

Update: ##/##/##

## Indice

* [Descripcion del proyecto](#descripcion-del-proyecto)
* [Estado del Proyecto](#estado-del-proyecto)
* [Tecnologias Utilizadas](#tecnologias-utilizadas)
* [Libreria Interna](#-libreria-interna)
* [Licecia](#licencia)
* [Versiones](#Versiones)
* [Codigo](#Codigo)

## Descripcion del proyecto
Script para automatizar la creacion de formularios de google para evaluacion docente.

El desarrollo de esta aplicacion se dio porque las fechas de evaluacion docente se habian acercado y el proyecto principal del mismo aun no estaba terminado, por lo que fue necesario generar un script para automatizar la generarcion de formularios de google que nos permitieran crear alli la evaluacion mientras se continuaba con el proyecto.

Dentro vienen ejemplos para hacer funcionar el script.

## Estado del Proyecto
Este Script se cuentra pausado.

## Tecnologias Utilizadas
* Python 3.11.5
* google-api-core          2.14.0
* google-api-python-client 2.108.0
* google-auth              2.23.4
* google-auth-httplib2     0.1.1
* google-auth-oauthlib     1.1.0
* googleapis-common-protos 1.61.0
* oauth2client             4.1.3
* oauthlib                 3.2.2
* httplib2                 0.22.0

## Licecia
Este Proyecto esta bajo Creative Commons CC-BY "Atribución 3.0 No portada" License:
https://creativecommons.org/licenses/by/3.0/deed.es

## Versiones

v0.1
--
Genera un formulario de google

## Codigo

```python
#---------------------------------------------------------------------------------------
# Script generador de formularios para evaluacion docente
#Name: EvalDoc-FormGen
#Version: 0.0.1
#Dev: Ing. Hector Rafael Gonzalez Vega
#Date: 21/11/2023
#update: ##/##/####
#---------------------------------------------------------------------------------------
from apiclient import discovery
from httplib2 import Http
from oauth2client import client, file, tools


def teacher_dicc(_list):
    '''
        Funcion para generar un diccionario que contendra los nombres de los profesores a evaluar.

        Args:
            _list (list): Lista con profesores a evaluar.
        
        Returns:
            dicc: Diccionario en formato para generar el formulario.
    '''
    teachers = []

    for _item in _list:
        teachers.append({
            'rowQuestion':{
                'title':_item
            }
        })

    return teachers
    
def answers_dicc(_list):
    '''
        Funcion para generar un diccionario que con las preguntas de la evaluacion.

        Args:
            _list (list): Lista con profesores a evaluar.
        
        Returns:
            dicc: Diccionario en formato para generar el formulario.
    '''
    answers = []
    for _item in _list:
        answers.append(
        {'value':_item}
        )    
    return answers

def generate_form(title, form_title, questions, teachers, answers):
    '''
        Funcion para generar un diccionario que con las preguntas de la evaluacion.

        Args:
            _list (list): Lista con profesores a evaluar.
        
        Returns:
            dicc: Diccionario en formato para generar el formulario.
    '''

    SCOPES = "https://www.googleapis.com/auth/forms.body"
    DISCOVERY_DOC = "https://forms.googleapis.com/$discovery/rest?version=v1"

    request = []

    id=0

    #Es necesario obtener los tokens de google api
    store = file.Storage("token.json")
    creds = None
    if not creds or creds.invalid:
        flow = client.flow_from_clientsecrets("client_secret.json", SCOPES)
        creds = tools.run_flow(flow, store)

    form_service = discovery.build(
        "forms",
        "v1",
        http=creds.authorize(Http()),
        discoveryServiceUrl=DISCOVERY_DOC,
        static_discovery=False,
    )

    # Cuerpo para crear el formulario
    NEW_FORM = {
        "info": {
            "title": title, # Titulo de la encuesta
            "documentTitle":form_title, # Titulo del documento
        }
    }

    teachers_dic = teacher_dicc(teachers)

    answers = answers_dicc(answers)

    for question in questions: # Preguntas
        request.append(
            {
                "createItem": {
                    "item": {
                        "title": (
                            question
                        ),
                        "questionGroupItem":{
                            "questions":teachers_dic,
                            'grid':{
                                'columns':{
                                    'type':'RADIO',
                                    'options':answers
                                }
                            }
                        }
                    },
                    "location": {"index": id},
                }
            },
        )
        id+=1

    for teacher in teachers: # Profesores
        coments = 'Comentarios para: '+teacher
        request.append(
            {
                "createItem": {
                    "item": {
                        "title": (
                            coments
                        ),
                        "questionItem":{
                            "question":{
                                'required':True,
                                'textQuestion':{
                                    'paragraph':True
                                }
                            },
                        }
                    },
                    "location": {"index": id},
                }
            },
        )
        id+=1

    # Cuerpo para agregar una pregunta de opcion multiple
    NEW_QUESTION = {
        "requests": request
    }

    # Se crea el formulario principal
    result = form_service.forms().create(body=NEW_FORM).execute()

    # Imprime el resultado del formulario que fue agregado
    get_result = form_service.forms().get(formId=result["formId"]).execute()
    print(get_result)



#---------------------EJEMPLO

#--Preguntas
#questions = [
#    'Pregunta 1',
#    'Pregunta 2',
#    'Pregunta 3',
#    ]

#--Respuestas
#answers = ['Desacuerdo','Mas o menos','De acuerdo']

#--Titulo para identificar el formulario
#title = 'Evaluación docente'

#--Titulo del formulario
#form_title = 'Evaluación docente Ago-Dic 2023'

#--Profesores a evaluar
#teachers = ['Alvarado','Flores','Guevara']

#--Llamada de la funcion
#generate_form(title, form_title, questions, teachers, answers)

```
