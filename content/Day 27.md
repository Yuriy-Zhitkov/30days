# Day 27

# **Build a draggable and resizable dashboard with Streamlit Elements**

Streamlit Elements is a third-party component made by [okld](https://github.com/okld) that gives you the tools to compose beautiful applications and dashboards with Material UI widgets, Monaco editor (Visual Studio Code), Nivo charts, and more.

## **How to use?**

### **Installation**

The first step is to install Streamlit Elements in your environment:

`pip install streamlit-elements==0.1.*`

It is recommended to pin the version to `0.1.*`, as future versions might introduce breaking API changes.

### **Usage**

You can refer to [Streamlit Elements README](https://github.com/okld/streamlit-elements#getting-started) for an in-depth usage guide.

## **What are we building?**

The goal of today's challenge is to create a dashboard composed of three Material UI cards:

- A first card with a Monaco code editor to input some data ;
- A second card to display that data in a Nivo Bump chart ;
- A third card to show a YouTube video URL defined with a `st.text_input`.

You can use data generated from Nivo Bump demo there, in 'data' tab: [https://nivo.rocks/bump/](https://nivo.rocks/bump/).

## **Demo app**

[https://camo.githubusercontent.com/767be70c92254555bd347ab07908fec67854c2264b77702581bd230fd7eac54f/68747470733a2f2f7374617469632e73747265616d6c69742e696f2f6261646765732f73747265616d6c69745f62616467655f626c61636b5f77686974652e737667](https://camo.githubusercontent.com/767be70c92254555bd347ab07908fec67854c2264b77702581bd230fd7eac54f/68747470733a2f2f7374617469632e73747265616d6c69742e696f2f6261646765732f73747265616d6c69745f62616467655f626c61636b5f77686974652e737667)

## **Code with line-by-line explanation**

```
# First, we will need the following imports for our application.

import json
import streamlit as st
from pathlib import Path

# As for Streamlit Elements, we will need all these objects.
# All available objects and there usage are listed there: https://github.com/okld/streamlit-elements#getting-started

from streamlit_elements import elements, dashboard, mui, editor, media, lazy, sync, nivo

# Change page layout to make the dashboard take the whole page.

st.set_page_config(layout="wide")

with st.sidebar:
    st.title("🗓️ #30DaysOfStreamlit")
    st.header("Day 27 - Streamlit Elements")
    st.write("Build a draggable and resizable dashboard with Streamlit Elements.")
    st.write("---")

    # Define URL for media player.
    media_url = st.text_input("Media URL", value="https://www.youtube.com/watch?v=vIQQR_yq-8I")

# Initialize default data for code editor and chart.
#
# For this tutorial, we will need data for a Nivo Bump chart.
# You can get random data there, in tab 'data': https://nivo.rocks/bump/
#
# As you will see below, this session state item will be updated when our
# code editor change, and it will be read by Nivo Bump chart to draw the data.

if "data" not in st.session_state:
    st.session_state.data = Path("data.json").read_text()

# Define a default dashboard layout.
# Dashboard grid has 12 columns by default.
#
# For more information on available parameters:
# https://github.com/react-grid-layout/react-grid-layout#grid-item-props

layout = [
    # Editor item is positioned in coordinates x=0 and y=0, and takes 6/12 columns and has a height of 3.
    dashboard.Item("editor", 0, 0, 6, 3),
    # Chart item is positioned in coordinates x=6 and y=0, and takes 6/12 columns and has a height of 3.
    dashboard.Item("chart", 6, 0, 6, 3),
    # Media item is positioned in coordinates x=0 and y=3, and takes 6/12 columns and has a height of 4.
    dashboard.Item("media", 0, 2, 12, 4),
]

# Create a frame to display elements.

with elements("demo"):

    # Create a new dashboard with the layout specified above.
    #
    # draggableHandle is a CSS query selector to define the draggable part of each dashboard item.
    # Here, elements with a 'draggable' class name will be draggable.
    #
    # For more information on available parameters for dashboard grid:
    # https://github.com/react-grid-layout/react-grid-layout#grid-layout-props
    # https://github.com/react-grid-layout/react-grid-layout#responsive-grid-layout-props

    with dashboard.Grid(layout, draggableHandle=".draggable"):

        # First card, the code editor.
        #
        # We use the 'key' parameter to identify the correct dashboard item.
        #
        # To make card's content automatically fill the height available, we will use CSS flexbox.
        # sx is a parameter available with every Material UI widget to define CSS attributes.
        #
        # For more information regarding Card, flexbox and sx:
        # https://mui.com/components/cards/
        # https://mui.com/system/flexbox/
        # https://mui.com/system/the-sx-prop/

        with mui.Card(key="editor", sx={"display": "flex", "flexDirection": "column"}):

            # To make this header draggable, we just need to set its classname to 'draggable',
            # as defined above in dashboard.Grid's draggableHandle.

            mui.CardHeader(title="Editor", className="draggable")

            # We want to make card's content take all the height available by setting flex CSS value to 1.
            # We also want card's content to shrink when the card is shrinked by setting minHeight to 0.

            with mui.CardContent(sx={"flex": 1, "minHeight": 0}):

                # Here is our Monaco code editor.
                #
                # First, we set the default value to st.session_state.data that we initialized above.
                # Second, we define the language to use, JSON here.
                #
                # Then, we want to retrieve changes made to editor's content.
                # By checking Monaco documentation, there is an onChange property that takes a function.
                # This function is called everytime a change is made, and the updated content value is passed in
                # the first parameter (cf. onChange: https://github.com/suren-atoyan/monaco-react#props)
                #
                # Streamlit Elements provide a special sync() function. This function creates a callback that will
                # automatically forward its parameters to Streamlit's session state items.
                #
                # Examples
                # --------
                # Create a callback that forwards its first parameter to a session state item called "data":
                # >>> editor.Monaco(onChange=sync("data"))
                # >>> print(st.session_state.data)
                #
                # Create a callback that forwards its second parameter to a session state item called "ev":
                # >>> editor.Monaco(onChange=sync(None, "ev"))
                # >>> print(st.session_state.ev)
                #
                # Create a callback that forwards both of its parameters to session state:
                # >>> editor.Monaco(onChange=sync("data", "ev"))
                # >>> print(st.session_state.data)
                # >>> print(st.session_state.ev)
                #
                # Now, there is an issue: onChange is called everytime a change is made, which means everytime
                # you type a single character, your entire Streamlit app will rerun.
                #
                # To avoid this issue, you can tell Streamlit Elements to wait for another event to occur
                # (like a button click) to send the updated data, by wrapping your callback with lazy().
                #
                # For more information on available parameters for Monaco:
                # https://github.com/suren-atoyan/monaco-react
                # https://microsoft.github.io/monaco-editor/api/interfaces/monaco.editor.IStandaloneEditorConstructionOptions.html

                editor.Monaco(
                    defaultValue=st.session_state.data,
                    language="json",
                    onChange=lazy(sync("data"))
                )

            with mui.CardActions:

                # Monaco editor has a lazy callback bound to onChange, which means that even if you change
                # Monaco's content, Streamlit won't be notified directly, thus won't reload everytime.
                # So we need another non-lazy event to trigger an update.
                #
                # The solution is to create a button that fires a callback on click.
                # Our callback doesn't need to do anything in particular. You can either create an empty
                # Python function, or use sync() with no argument.
                #
                # Now, everytime you will click that button, onClick callback will be fired, but every other
                # lazy callbacks that changed in the meantime will also be called.

                mui.Button("Apply changes", onClick=sync())

        # Second card, the Nivo Bump chart.
        # We will use the same flexbox configuration as the first card to auto adjust the content height.

        with mui.Card(key="chart", sx={"display": "flex", "flexDirection": "column"}):

            # To make this header draggable, we just need to set its classname to 'draggable',
            # as defined above in dashboard.Grid's draggableHandle.

            mui.CardHeader(title="Chart", className="draggable")

            # Like above, we want to make our content grow and shrink as the user resizes the card,
            # by setting flex to 1 and minHeight to 0.

            with mui.CardContent(sx={"flex": 1, "minHeight": 0}):

                # This is where we will draw our Bump chart.
                #
                # For this exercise, we can just adapt Nivo's example and make it work with Streamlit Elements.
                # Nivo's example is available in the 'code' tab there: https://nivo.rocks/bump/
                #
                # Data takes a dictionary as parameter, so we need to convert our JSON data from a string to
                # a Python dictionary first, with `json.loads()`.
                #
                # For more information regarding other available Nivo charts:
                # https://nivo.rocks/

                nivo.Bump(
                    data=json.loads(st.session_state.data),
                    colors={ "scheme": "spectral" },
                    lineWidth=3,
                    activeLineWidth=6,
                    inactiveLineWidth=3,
                    inactiveOpacity=0.15,
                    pointSize=10,
                    activePointSize=16,
                    inactivePointSize=0,
                    pointColor={ "theme": "background" },
                    pointBorderWidth=3,
                    activePointBorderWidth=3,
                    pointBorderColor={ "from": "serie.color" },
                    axisTop={
                        "tickSize": 5,
                        "tickPadding": 5,
                        "tickRotation": 0,
                        "legend": "",
                        "legendPosition": "middle",
                        "legendOffset": -36
                    },
                    axisBottom={
                        "tickSize": 5,
                        "tickPadding": 5,
                        "tickRotation": 0,
                        "legend": "",
                        "legendPosition": "middle",
                        "legendOffset": 32
                    },
                    axisLeft={
                        "tickSize": 5,
                        "tickPadding": 5,
                        "tickRotation": 0,
                        "legend": "ranking",
                        "legendPosition": "middle",
                        "legendOffset": -40
                    },
                    margin={ "top": 40, "right": 100, "bottom": 40, "left": 60 },
                    axisRight=None,
                )

        # Third element of the dashboard, the Media player.

        with mui.Card(key="media", sx={"display": "flex", "flexDirection": "column"}):
            mui.CardHeader(title="Media Player", className="draggable")
            with mui.CardContent(sx={"flex": 1, "minHeight": 0}):

                # This element is powered by ReactPlayer, it supports many more players other
                # than YouTube. You can check it out there: https://github.com/cookpete/react-player#props

                media.Player(url=media_url, width="100%", height="100%", controls=True)
```

## **Any question?**

Feel free to ask any question regarding Streamlit Elements or this challenge there: [Streamlit Elements Topic](https://discuss.streamlit.io/t/streamlit-elements-build-draggable-and-resizable-dashboards-with-material-ui-nivo-charts-and-more/24616)

# **Создайте перетаскиваемую панель инструментов с изменяемым размером с помощью Streamlit Elements**

Streamlit Elements—это сторонний компонент который создал [okld](https://github.com/okld) чтобы вы могли делать красивые приложения и информационные панели с виджетами Material UI, редактором Monaco (Visual Studio Code), диаграммами Nivo, и так далее.

## **Как использовать?**

### **Монтаж**

Первым шагом является установка Streamlit Elements в вашей среде:

`pip install streamlit-elements==0.1.*`

Рекомендуется закрепить версию `0.1.*`, так как в будущих версиях могут быть внесены критические изменения API.

### **Применение**

Подробное руководство по использованию можно найти в [README Streamlit Elements](https://github.com/okld/streamlit-elements#getting-started).

## **Что мы строим?**

Цель сегодняшнего задания—создать панель управления, состоящую из трех карт пользовательского интерфейса материалов:

- Первая карта с редактором кода Monaco для ввода некоторых данных;
- Вторая карта для отображения этих данных в диаграмме Nivo Bump;
- Третья карта для отображения URL-адреса видео YouTube, определенного с помощью `st.text_input`.

Вы можете использовать данные, сгенерированные из демонстрации Nivo Bump, на вкладке «данные»: [https://nivo.rocks/bump/](https://nivo.rocks/bump/).

## **Демонстрационное приложение**

[https://camo.githubusercontent.com/767be70c92254555bd347ab07908fec67854c2264b77702581bd230fd7eac54f/68747470733a2f2f7374617469632e73747265616d6c69742e696f2f6261646765732f73747265616d6c69745f62616467655f626c61636b5f77686974652e737667](https://camo.githubusercontent.com/767be70c92254555bd347ab07908fec67854c2264b77702581bd230fd7eac54f/68747470733a2f2f7374617469632e73747265616d6c69742e696f2f6261646765732f73747265616d6c69745f62616467655f626c61636b5f77686974652e737667)

## **Код с построчными пояснениями**

```
# Сначала нам понадобятся следующие импорты для нашего приложения.

import json
import streamlit as st
from pathlib import Path

# Что касается элементов Streamlit, то нам понадобятся все эти объекты.
# Здесь перечислены все доступные объекты и их использование: https://github.com/okld/streamlit-elements#getting-started

from streamlit_elements import elements, dashboard, mui, editor, media, lazy, sync, nivo

# Измените макет страницы, чтобы панель инструментов занимала всю страницу.

st.set_page_config(layout="wide")

with st.sidebar:
    st.title("🗓️ #30DaysOfStreamlit")
    st.header("Day 27 - Streamlit Elements")
    st.write("Build a draggable and resizable dashboard with Streamlit Elements.")
    st.write("---")

		# Определите URL для медиаплеера.
    media_url = st.text_input("Media URL", value="https://www.youtube.com/watch?v=vIQQR_yq-8I")

# Initialize default data for code editor and chart.
#
# For this tutorial, we will need data for a Nivo Bump chart.
# You can get random data there, in tab 'data': https://nivo.rocks/bump/
#
# As you will see below, this session state item will be updated when our
# code editor change, and it will be read by Nivo Bump chart to draw the data.

# Инициализируйте данные по умолчанию для редактора кода и диаграммы.
#
# Для этого урока нам понадобятся данные для диаграммы Nivo Bump.
# Вы можете получить случайные данные во вкладке «данные»: https://nivo.rocks/bump/
#
# Как вы увидите ниже, этот элемент session state будет обновляться, когда наш
# редактор кода изменится, и он будет прочитан диаграммой Nivo Bump для извлечения данных.

if "data" not in st.session_state:
    st.session_state.data = Path("data.json").read_text()

# Определите макет панели инструментов по умолчанию.
# Сетка панели инструментов по умолчанию имеет 12 столбцов.
#
# Для дополнительной информации о доступных параметрах:
# https://github.com/react-grid-layout/react-grid-layout#grid-item-props

layout = [
		# Элемент редактора расположен в координатах x=0 и y=0, занимает 6/12 столбцов и имеет высоту 3.
    dashboard.Item("editor", 0, 0, 6, 3),
		# Элемент диаграммы расположен в координатах x=6 и y=0, занимает 6/12 столбцов и имеет высоту 3.
    dashboard.Item("chart", 6, 0, 6, 3),
    # Элемент мультимедиа расположен в координатах x=0 и y=3, занимает 6/12 столбцов и имеет высоту 4.
    dashboard.Item("media", 0, 2, 12, 4),
]

# Создайте рамку для отображения элементов.

with elements("demo"):

		 # Создайте новую панель мониторинга с указанным выше макетом.
     #
     # draggableHandle — это селектор запросов CSS для определения перетаскиваемой части каждого элемента панели.
     # Здесь элементы с именем класса "перетаскиваемый" будут перетаскиваемыми.
     #
     # Для получения дополнительной информации о доступных параметрах сетки панели инструментов:
     # https://github.com/react-grid-layout/react-grid-layout#grid-layout-props
     # https://github.com/react-grid-layout/react-grid-layout#responsive-grid-layout-props

    with dashboard.Grid(layout, draggableHandle=".draggable"):

				 # Первая карта, редактор кода.
         #
         # Мы используем параметр 'key' для определения правильного элемента панели.
         #
         # Чтобы содержимое карты автоматически заполняло доступную высоту, мы будем использовать CSS flexbox.
         # sx — это параметр, доступный в каждом виджете Material UI для определения атрибутов CSS.
         #
         # Для получения дополнительной информации о Card, flexbox и sx:
         # https://mui.com/components/cards/
         # https://mui.com/system/flexbox/
         # https://mui.com/system/the-sx-prop/

        with mui.Card(key="editor", sx={"display": "flex", "flexDirection": "column"}):

						# Чтобы сделать этот заголовок перетаскиваемым, нам просто нужно установить для его имени класса значение «перетаскиваемый»,
						# как определено выше в draggableHandle панели инструментов.

            mui.CardHeader(title="Editor", className="draggable")

            # We want to make card's content take all the height available by setting flex CSS value to 1.
            # We also want card's content to shrink when the card is shrinked by setting minHeight to 0.

						# Мы хотим, чтобы содержимое карточки занимало всю доступную высоту, установив для flex CSS значение 1.
            # Мы также хотим, чтобы содержимое карты сжималось, когда карта сжимается, установив для minHeight значение 0.

            with mui.CardContent(sx={"flex": 1, "minHeight": 0}):

								# Вот наш редактор кода Monaco.
                #
                # Во-первых, мы устанавливаем значение по умолчанию st.session_state.data, которое мы инициализировали выше.
                # Во-вторых, мы определяем используемый язык, здесь JSON.
                #
                # Затем мы хотим получить изменения, внесенные в содержимое редактора.
                # Проверяя документацию Monaco, мы обнаруживаем свойство onChange, которое принимает функцию.
                # Эта функция вызывается каждый раз при внесении изменений, и обновленное значение содержимого передается в
                # первый параметр (см. onChange: https://github.com/suren-atoyan/monaco-react#props)
                #
                # Элементы Streamlit предоставляют специальную функцию sync(). Эта функция создает обратный вызов, который
                # автоматически пересылает свои параметры элементам session state Streamlit.
                #
                # Примеры
                # --------
                # Создайте обратный вызов, который перенаправляет свой первый параметр в элемент session state с именем «data»:
                # >>> editor.Monaco(onChange=sync("data"))
                # >>> print(st.session_state.data)
                #
                # Создайте обратный вызов, который перенаправляет свой второй параметр элементу session state с именем «ev»:
                # >>> editor.Monaco(onChange=sync(None, "ev"))
                # >>> print(st.session_state.ev)
                #
                # Создайте обратный вызов, который перенаправляет оба своих параметра в session state:
                # >>> editor.Monaco(onChange=sync("data", "ev"))
                # >>> print(st.session_state.data)
                # >>> print(st.session_state.ev)
                #
                # Теперь есть проблема: onChange вызывается каждый раз, когда вносятся изменения, что означает, что каждый раз когда
                # вы вводите один символ, все ваше приложение Streamlit запускается повторно.
                #
                # Чтобы избежать этой проблемы, вы можете указать Streamlit Elements ждать другого события
                # (как например, нажатие кнопки), чтобы отправить обновленные данные, обернув ваш обратный вызов с помощью lazy().
                #
                # Для получения дополнительной информации о доступных параметрах для Монако:
                # https://github.com/suren-atoyan/monaco-react
                # https://microsoft.github.io/monaco-editor/api/interfaces/monaco.editor.ISandaloneEditorConstructionOptions.html

                editor.Monaco(
                    defaultValue=st.session_state.data,
                    language="json",
                    onChange=lazy(sync("data"))
                )

            with mui.CardActions:

								# В редакторе Monaco есть ленивый обратный вызов функции, связанный с onChange, что означает, что даже если вы измените
                 # Контент Монако, Streamlit не будет получать уведомления напрямую, поэтому не будет перезагружаться каждый раз.
                 # Итак, нам нужно еще одно неленивое событие для запуска обновления.
                 #
                 # Наше решение будет в создании кнопки, которая запускает обратный вызов при нажатии.
                 # Наш обратный вызов не требует особых действий. Вы можете либо создать пустой
                 # Функция Python или использовать sync() без аргументов.
                 #
                 # Теперь каждый раз, когда вы будете нажимать эту кнопку, будет запускаться обратный вызов onClick, но и все остальные
                 # ленивые обратные вызовы, которые за это время изменились, также будут вызваны.

                mui.Button("Apply changes", onClick=sync())

        # Second card, the Nivo Bump chart.
        # We will use the same flexbox configuration as the first card to auto adjust the content height.

				# Вторая карта, диаграмма Nivo Bump.
        # Мы будем использовать ту же конфигурацию flexbox, которую мы использовали на первая карте, для автоматической настройки высоты контента.

        with mui.Card(key="chart", sx={"display": "flex", "flexDirection": "column"}):

            # To make this header draggable, we just need to set its classname to 'draggable',
            # as defined above in dashboard.Grid's draggableHandle.

						# Чтобы сделать этот заголовок перетаскиваемым, нам просто нужно установить для его имени класса значение «перетаскиваемый»,
            # как определено выше в draggableHandle панели инструментов.

            mui.CardHeader(title="Chart", className="draggable")

            # Like above, we want to make our content grow and shrink as the user resizes the card,
            # by setting flex to 1 and minHeight to 0.

						# Как мы обсуждали раньше, нам нужно чтобы наш контент увеличивался и уменьшался по мере того, как пользователь менял размер карты,
            # установливая значение flex к 1 и minHeight к 0.

            with mui.CardContent(sx={"flex": 1, "minHeight": 0}):

								# Здесь мы будем рисовать нашу диаграмму Bump.
                #
                # Для этого упражнения мы можем просто адаптировать пример Nivo чтобы оно работалои с Streamlit Elements.
                # Пример Nivo доступен на вкладке «код» здесь: https://nivo.rocks/bump/
                #
                # Данные принимают словарь в качестве параметра, поэтому нам нужно преобразовать наши данные JSON из строки в
                 # словарь Python с `json.loads()`.
                 #
                 # Для получения дополнительной информации о других доступных графиках Nivo:
                 # https://nivo.rocks/

                nivo.Bump(
                    data=json.loads(st.session_state.data),
                    colors={ "scheme": "spectral" },
                    lineWidth=3,
                    activeLineWidth=6,
                    inactiveLineWidth=3,
                    inactiveOpacity=0.15,
                    pointSize=10,
                    activePointSize=16,
                    inactivePointSize=0,
                    pointColor={ "theme": "background" },
                    pointBorderWidth=3,
                    activePointBorderWidth=3,
                    pointBorderColor={ "from": "serie.color" },
                    axisTop={
                        "tickSize": 5,
                        "tickPadding": 5,
                        "tickRotation": 0,
                        "legend": "",
                        "legendPosition": "middle",
                        "legendOffset": -36
                    },
                    axisBottom={
                        "tickSize": 5,
                        "tickPadding": 5,
                        "tickRotation": 0,
                        "legend": "",
                        "legendPosition": "middle",
                        "legendOffset": 32
                    },
                    axisLeft={
                        "tickSize": 5,
                        "tickPadding": 5,
                        "tickRotation": 0,
                        "legend": "ranking",
                        "legendPosition": "middle",
                        "legendOffset": -40
                    },
                    margin={ "top": 40, "right": 100, "bottom": 40, "left": 60 },
                    axisRight=None,
                )

				# Третий элемент панели управления, медиаплеер.

        with mui.Card(key="media", sx={"display": "flex", "flexDirection": "column"}):
            mui.CardHeader(title="Media Player", className="draggable")
            with mui.CardContent(sx={"flex": 1, "minHeight": 0}):

								# Этот элемент получает энергию от ReactPlayer и поддерживает гораздо больше других плееров
                # чем YouTube. Вы можете проверить это здесь: https://github.com/cookpete/react-player#props.

                media.Player(url=media_url, width="100%", height="100%", controls=True)
```