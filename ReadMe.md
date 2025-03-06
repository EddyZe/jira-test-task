### Начальные задания по написанию скриптов в плагине Scriptrunner

---

1.
```groovy
'Hello world'
```

----
2.
```groovy
Users.getLoggedInUser().getDisplayName()
```
---
3.
```groovy
import com.atlassian.jira.component.ComponentAccessor

def issueKey = "TES1-1"
def issue = Issues.getByKey(issueKey)

String comment = "test comment"

def manager = ComponentAccessor.commentManager

manager.create(issue, Users.getLoggedInUser(), comment, true)
```

---

4.
```groovy
def projectKey = "Test-task"
Issues.search("project = ${projectKey} AND status != Closed AND status != Resolved").each {q -> 
    q
 }
```
---
5.
```groovy
Issues.getByKey('SS-24').transition('In progress') {
    setAssignee('edzeeeee')
    setComment('Issue progress has started')
}
```
---
6.
```groovy
import com.atlassian.jira.issue.IssueInputParametersImpl
import com.atlassian.jira.issue.Issue
import com.atlassian.jira.bean.SubTask
import com.atlassian.jira.component.ComponentAccessor

def issue = event.issue
def changeLog = event.changeLog
def user = Users.getLoggedInUser()

if (issue.issueType.isSubTask()) {
    return
}

if (issue.getStatus().simpleStatus.getStatusCategory().getName() == "In Progress") {
    return
}

createSubtask(issue)


def createSubtask(Issue parentIssue) {
    def subTaskManager = ComponentAccessor.subTaskManager
    def asUser = Users.getLoggedInUser()
    def constantsManager = ComponentAccessor.constantsManager
    def issueService = ComponentAccessor.issueService

    def subtaskIssueType = constantsManager.allIssueTypeObjects.findByName('Sub-task')

    assert subtaskIssueType?.subTask

    def issueInputParameters = new IssueInputParametersImpl()
    issueInputParameters
            .setProjectId(parentIssue.projectId)
            .setIssueTypeId(subtaskIssueType.id)
            .setSummary('A new subtask')
            .setDescription('A description')
            .setReporterId(asUser.username)

    def createValidationResult = ComponentAccessor.issueService.validateSubTaskCreate(asUser, parentIssue.id, issueInputParameters)
    if (!createValidationResult.valid) {
        log.error createValidationResult.errorCollection
        return
    }

    def newIssue = issueService.create(asUser, createValidationResult).issue
    subTaskManager.createSubTaskIssueLink(parentIssue, newIssue, asUser)
}

```

----

7.
```groovy

import com.atlassian.jira.component.ComponentAccessor


def issue = Issues.getByKey("SS-26") 


def issueLinkManager = ComponentAccessor.issueLinkManager
def linkedIssues = issueLinkManager.getIssueLinks(issue.getId())

def description = issue.description ?: "Нет описания" 
description += "\n\nСвязанные задачи:\n"
linkedIssues.each { linkedIssue ->
    description += linkedIssue.getSourceObject().getEpicName() + " " + linkedIssue.getDestinationObject().getDescription()
} 

issue.update { 
    setAssignee(Users.getLoggedInUser())
    setDescription(description)
 }

log.info("Описание задачи ${issue.key} обновлено с информацией о связанных задачах.")
```

### Работа со Scriptrunner Behaviors 


```groovy
import com.atlassian.jira.component.ComponentAccessor
import com.onresolve.jira.behaviours.types.FieldOption
import java.util.List
import com.onresolve.jira.groovy.user.FormField
import java.time.LocalDateTime
import java.time.format.DateTimeFormatter

def checkBox = getFieldByName("Checkbox-Radiobutton")
def dataTimePieck = getFieldByName("Data time picker")
def labelField = getFieldByName("Label Field")
def miltiSelectList = getFieldByName("Multi Select List")
def multiTextField = getFieldByName("Multi text field")
def radioButtons = getFieldByName("Radio Button")
def selectList = getFieldByName("Select list")
def selectLitField = getFieldByName("Select list field")
def dataPicker = getFieldByName("data picker")
def userPicker = getFieldByName("User picker")
def singleIssuePicker = getFieldByName("Single Issue Picker")
def textField = getFieldByName("Text Field")
def versionPicker = getFieldByName("Version picker single")
def userPcikerMulti = getFieldByName("User picker multi")

def cascadeList = getFieldByName("Cascade List")
def values = cascadeList.value as List

if (values) {

    def currentTab = 0
    def parentValue = values.get(0)
    def childValue = ""

    if (values.size() > 1) {
        childValue = values.get(1)
    }

    if (parentValue == "Dates") {
        showDate(dataTimePieck, dataPicker, childValue)
    } else {
        dataTimePieck.setHidden(true)
        dataPicker.setHidden(true)
    }

    if (parentValue == "Checkbox-Radiobutton") {
        showRadioButtons(radioButtons, checkBox, childValue)
    } else {
        radioButtons.setHidden(true)
        checkBox.setHidden(true)
        checkBox.setRequired(false)
        radioButtons.setReadOnly(false)
    }

    if (parentValue == "Select list") {
        showSelectFields(selectList, miltiSelectList, childValue)
    } else {
        selectList.setHidden(true)
        miltiSelectList.setHidden(true)
    }

    if (parentValue == "Text fields") {
        showTextField(textField, labelField, multiTextField, childValue)
    } else {
        textField.setHidden(true)
        labelField.setHidden(true)
        multiTextField.setHidden(true)
    }

    if (parentValue == "User Pickers") {
        showUserPickerFields(userPcikerMulti, userPicker, childValue)
    } else {
        userPicker.setHidden(true)
        userPcikerMulti.setHidden(true)
    }

    if (parentValue == "Tabs") {
        currentTab = editTabs(childValue, currentTab)
    }

    if (parentValue == "Special fields") {
        showSpecialFields(singleIssuePicker, versionPicker, childValue)

    } else {
        singleIssuePicker.setHidden(true)
        versionPicker.setHidden(true)
    }

}

Integer editTabs(Object childValue, Integer currentTab) {
    switch (childValue) {
        case "Следующая вкладка":
            currentTab++

            if (currentTab > 2) {
                currentTab = 0
            }

            switchTab(currentTab)
            break
        case "Предыдущая вкладка":
            currentTab--;

            if (currentTab < 0) {
                currentTab = 2
            }
            switchTab(currentTab)
            break
        case "Спрятать вкладки":
            for (int i = 1; i < 3; i++) {
                hideTab(i)
            }
            break
        case "Показать все вкладки":
            for (int i = 0; i < 3; i++) {
                showTab(i)
            }
            break
    }

    return currentTab
}

void showSpecialFields(FormField singleIssuePicker, FormField versionPicker, Object childValue) {
    singleIssuePicker.setHidden(false)
    versionPicker.setHidden(false)

    switch (childValue) {
        case "Поставить значения":

            singleIssuePicker.setFormValue("SS-26")
            versionPicker.setFormValue("Version 1")
            break
        case "Почистить значения":
            singleIssuePicker.setFormValue([])
            versionPicker.setFormValue(null)
            break
    }
}

void showUserPickerFields(FormField userPcikerMulti, FormField userPicker, Object childValue) {
    userPicker.setHidden(false)
    userPcikerMulti.setHidden(false)

    switch (childValue) {
        case "Поставить значения":
            userPicker.setFormValue("test")
            userPcikerMulti.setFormValue([Users.getLoggedInUser().getUsername(), "test"])
            break
        case "Поменять значения местами":
            def usersMultiPickerField = userPcikerMulti.value as List
            def userPickerField = userPicker.value as String

            if (usersMultiPickerField.size() > 0) {
                userPicker.setFormValue(usersMultiPickerField[0])
            }

            if (userPickerField != "") {
                userPcikerMulti.setFormValue(userPickerField)
            }

            break
        case "Почистить значения":
            userPicker.setFormValue("")
            userPcikerMulti.setFormValue([])
            break
    }
}

void showTextField(FormField textField, FormField labelField, FormField multiTextField, Object childValue) {
    textField.setHidden(false)
    labelField.setHidden(false)
    multiTextField.setHidden(false)

    switch (childValue) {
        case "Заполнить поля":
            textField.setFormValue('Пара слов')
            multiTextField.setFormValue('Вот пару\n строк')
            labelField.setFormValue(issueContext.getIssueType().name)
            break

        case "Проставить ошибки":
            textField.setError("А вот и ошибка со смешным текстом")
            multiTextField.setError("Вот еще одна ошибка со смешным текстом")
            labelField.setError("И вот последняя ошибка со смешным текстом")
            break
        case "Убрать ошибки":
            textField.setError("")
            multiTextField.setError("")
            labelField.setError("")
            break
        case "Почистить значения":
            textField.setFormValue('')
            multiTextField.setFormValue('')
            labelField.setFormValue("")
            break
        case "Поменять значения":
            def valText = textField.value as String
            def valMultiText = multiTextField.value as String
            def valLabel = labelField.value as String

            textField.setFormValue(valLabel)
            multiTextField.setFormValue(valText)
            labelField.setFormValue(valMultiText)

            break
    }
}

void showSelectFields(FormField selectList, FormField miltiSelectList, Object childValue) {
    selectList.setHidden(false)
    miltiSelectList.setHidden(false)

    switch (childValue) {
        case "Поставить значения":
            miltiSelectList.setFormValue(['Поставить значения', 'Добавить несуществующие опции', 'пятая опция'])
            selectList.setFormValue('Почистить значения')
            break
        case "Поменять значения местами":
            selectList.setFormValue('Почистить значения')
            miltiSelectList.setFormValue(['Поставить значения'])
            break
        case "Добавить несуществующие опции":
            def options = Map.of("None", "None", "testcustom2", "Несуществующие поле 2", "testcustom3", "Несуществующие поле 3")
            selectList.setFieldOptions(options)
            miltiSelectList.setFieldOptions(options)
            break
        case "Почистить значения":
            selectList.setFormValue(null)
            miltiSelectList.setFormValue(null)
            break
    }
}

void showRadioButtons(FormField radioButtons, FormField checkBox, Object childValue) {
    radioButtons.setHidden(false)
    checkBox.setHidden(false)

    switch (childValue) {
        case "Поставить значения":
            checkBox.setFormValue(['Поставить значения', 'Поменять опции', 'Еще одна 5 опция'])
            radioButtons.setFormValue('Почистить значения')
            break
        case "Обязательность и Только чтение":
            checkBox.setRequired(true)
            radioButtons.setReadOnly(true)
            break
        case "Поменять опции":

            break
        case "Почистить значения":
            checkBox.setFormValue([])
            radioButtons.setFormValue(null)
            break
    }
}

void showDate(FormField dataTimePieck, FormField dataPicker, Object childValue) {
    def dateFormat = "d/MMM/yy"
    def dateTimeFormat = "dd/MMM/yy h:mm a"

    dataTimePieck.setHidden(false)
    dataPicker.setHidden(false)

    def currentDate = LocalDateTime.now()

    switch (childValue) {
        case "Проставить текущие даты":
            dataTimePieck.setFormValue(currentDate.format(dateTimeFormat))
            dataPicker.setFormValue(currentDate.format(dateFormat))
            break
        case "+++":
            def dataPickerDate
            def dateTimePicker

            try {
                dataPickerDate = dataTimePieck.value as LocalDateTime
                dateTimePicker = dataPicker as LocalDateTime
            } catch (Exception e) {
                log.warn("Error parse date: " + e)
                dataPickerDate = currentDate
                dateTimePicker = currentDate
            }

            def formatter = DateTimeFormatter.ofPattern(dateFormat)
            def parsedDate = LocalDateTime.parse(dataPickerDate.toString())
            dataPicker.setFormValue(parsedDate.plusDays(3).format(dateFormat))

            formatter = DateTimeFormatter.ofPattern(dateTimeFormat)
            parsedDate = LocalDateTime.parse(parsedDate.toString())
            dataTimePieck.setFormValue(parsedDate.plusDays(3).plusMinutes(3).format(dateTimeFormat))

            break
        case "---":
            def dataPickerDate
            def dateTimePicker

            try {
                dataPickerDate = dataTimePieck.value as LocalDateTime
                dateTimePicker = dataPicker as LocalDateTime
            } catch (Exception e) {
                log.warn("Error parse date: " + e)
                dataPickerDate = currentDate
                dateTimePicker = currentDate
            }

            def formatter = DateTimeFormatter.ofPattern(dateFormat)
            def parsedDate = LocalDateTime.parse(dataPickerDate.toString())
            dataPicker.setFormValue(parsedDate.minusDays(3).format(dateFormat))

            formatter = DateTimeFormatter.ofPattern(dateTimeFormat)
            parsedDate = LocalDateTime.parse(parsedDate.toString())
            dataTimePieck.setFormValue(parsedDate.minusDays(1).minusHours(1).minusMinutes(1).format(dateTimeFormat))

            break
        case "Почистить значения":
            dataTimePieck.setFormValue("")
            dataPicker.setFormValue("")
            break
    }
}

```

### Работа со Scriptrunner Behaviors + REST api

---

1.

```groovy
def coin = getFieldByName("Coin")
def currency = getFieldByName("Currency")
def price = getFieldByName("Price")

def baseUrl = 'https://api.coingecko.com/api/v3'

def coinsResponse = new URL(baseUrl + "/coins/markets?vs_currency=usd&per_page=10").getText();
def currensyResponse = new URL(baseUrl + "/simple/supported_vs_currencies").getText()
def priceUrl = "/simple/price?ids=%s&vs_currencies=usd"

def coins = new JsonSlurper().parseText(coinsResponse.toString())
def currencys = new JsonSlurper().parseText(currensyResponse.toString()) as List

def coinOptions = new HashMap()
def currencyOption = new HashMap()
def priceOption = new HashMap()

coins.each { c -> 
    def coinFields = c as Map

    def symbol = coinFields.get("symbol")

    if (currencys.contains(symbol)) {
        coinOptions.put(coinFields.get("id"), coinFields.get("name"))
        priceOption.put(coinFields.get("id"), coinFields.get("current_price").toString())
    }
 }

 currencys.each { c ->
    currencyOption.put(c, c)
 }

coin.convertToSingleSelect()
coin.setFieldOptions(coinOptions)
 

currency.convertToSingleSelect()
currency.setFieldOptions(currencyOption)

price.convertToSingleSelect()
price.setFieldOptions(priceOption)
```

2.

```groovy
def coin = getFieldByName("Coin")
def currency = getFieldByName("Currency")
def price = getFieldByName("Price")
price.setReadOnly(true)

CoinGeckoApiClient client = new CoinGeckoApiClientImpl();

def coins = client.getCoinMarkets("usd")
def currencys = client.getSupportedVsCurrencies()

def coinOptions = new HashMap()
def currencyOption = new HashMap()
def priceOption = new HashMap()

coins.each { c -> 
    def symbol = c.getSymbol()

    if (currencys.contains(symbol)) {
        coinOptions.put(c.getId(), c.getName())
        priceOption.put(c.getId(), c.getCurrentPrice().toString())
    }
 }

 currencys.each { c ->
    currencyOption.put(c, c)
 }

coin.convertToSingleSelect()
coin.setFieldOptions(coinOptions)
 

currency.convertToSingleSelect()
currency.setFieldOptions(currencyOption)

price.convertToSingleSelect()
price.setFieldOptions(priceOption)
```
### JIRA api

---

1

```groovy
import com.atlassian.jira.component.ComponentAccessor
import com.atlassian.jira.issue.Issue
import com.atlassian.jira.project.Project


def projectKey = "SS"
def issueType = "User Task"

Issues.create(projectKey, issueType) {
    setSummary("Создание задачи с помощью скрипта")
    setDescription("descroption")
    setAssignee("edzeeeee")
    setReporter("test")
    setComponents("UI")
}
```

---

2

```groovy
import com.atlassian.jira.issue.MutableIssue
import com.atlassian.jira.event.type.EventDispatchOption
import com.atlassian.jira.component.ComponentAccessor
import com.atlassian.jira.issue.Issue

def issueKey = "SS-39"

def issueManager = ComponentAccessor.getIssueManager()

MutableIssue issue = issueManager.getIssueByCurrentKey(issueKey)
def user = issue.getReporter()
issue.setSummary("Новый заголовок")


issueManager.updateIssue(user, issue, EventDispatchOption.DO_NOT_DISPATCH, false)
```

3

```groovy

import com.atlassian.jira.component.ComponentAccessor
import com.atlassian.jira.issue.Issue

def issueKey = "SS-39"


def issue = Issues.getByKey(issueKey)
def issueService = ComponentAccessor.getIssueService()

def paramsIssue = issueService.newIssueInputParameters()
paramsIssue.summary = 'А вот еще один новый заголовк'
paramsIssue.description = "А вот новое описание"

def validationResult = issueService.validateUpdate(issue.getAssignee(), issue.getId(), paramsIssue)
if (validationResult.valid) {
    def updateResult = issueService.update(issue.getAssignee(), validationResult)
    if (!updateResult.errorCollection.hasAnyErrors()) {
        log.info("Задача обновлена")
    }
}
```

---

4

```groovy

```

