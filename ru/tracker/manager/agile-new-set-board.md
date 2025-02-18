# Как настроить доску

Чтобы перейти на страницу доски, на панели слева нажмите ![](../../_assets/tracker/svg/boards.svg)&nbsp;**{{ ui-key.startrek.ui_components_NavigationBar.boards }}** и выберите доску.

## Создать задачу {#create-task}

Новую задачу можно создать прямо на доске. Для этого достаточно указать ее название и очередь — остальные поля вы сможете заполнить позже.

1. Откройте страницу доски.

1. В крайней левой колонке нажмите значок ![](../../_assets/tracker/svg/add-task.svg) и выберите **{{ ui-key.startrek.ui_components_agile_agile-board_AgileBoardEmptyScreen.empty-screen--new-issue }}**.

1. Введите название задачи.

1. В выпадающем списке выберите очередь, в которой будет создана задача.

1. Нажмите кнопку **Создать** или клавишу **Enter**.

## Добавить задачу на доску {#add-task}

Добавленные на доску задачи отображаются в колонках, которые соответствуют статусу задач. Если добавленные задачи не отобразились на доске, убедитесь, что их статусы [привязаны к колонкам](agile-new-columns.md). Чтобы просмотреть список статусов, которые не используются на доске, рядом с крайней правой колонкой нажмите значок ![](../../_assets/tracker/svg/unused-status.svg).

{% note warning %}

На доску можно добавить не более 2000 задач.

{% endnote %}

### Со страницы доски {#from-board}

1. Откройте страницу доски.

1. В крайней левой колонке нажмите значок ![](../../_assets/tracker/svg/add-task.svg) и выберите **{{ ui-key.startrek.ui_components_CreateIssuePopup.existing-issue }}**.

1. Начните вводить ключ или название задачи и выберите подходящий вариант из списка. 

### Со страницы задачи {#from-ticket}

1. Откройте страницу задачи.

1. Нажмите на поле **{{ ui-key.startrek-backend.fields.issue.boards }}** на панели справа. Если поля **{{ ui-key.startrek-backend.fields.issue.boards }}** нет, добавьте его с помощью кнопки **{{ ui-key.startrek.ui_components_IssueSidebar.add-sidebar-field }}**.

1. Начните вводить название доски и выберите подходящий вариант из списка.

### Добавить несколько задач {#from-bulk}

1. Отберите нужные задачи с помощью [фильтра](../user/create-filter.md).

1. Выберите задачи, которые вы хотите добавить на доску.

1. На панели снизу нажмите ![](../../_assets/horizontal-ellipsis.svg) и выберите **{{ ui-key.startrek.ui_components_IssueBulkActionPanel.add-to-board }}**.

1. Начните вводить название доски и выберите подходящий вариант из списка. 

1. Дождитесь окончания обработки задач.

## Автодобавление задач {#autoadd}

Чтобы автоматически добавлять на доску задачи, например, все задачи очереди с определенным исполнителем, используйте автодобавление:

1. На доске нажмите **{{ ui-key.startrek.ui_components_PageAgileBoard_PageAgileBoardHeader.settings }}** → **{{ ui-key.startrek.ui_components_agile_settings_AgileSettingsLayout.issue-updates }}**.
1. В блоке **{{ ui-key.startrek.ui_components_agile_settings_BoardIssueUpdatesSettings.import-title }}** нажмите **+ {{ ui-key.startrek.ui_components_issues-import_IssuesImportFilters.add-parameters }}**. Выберите из списка параметр и укажите значение. Например, чтобы добавить на доску все задачи, в которых вы являетесь исполнителем, выберите параметр **Исполнитель** и укажите для него значение **Я**. Чтобы добавить дополнительные параметры, повторите этот шаг.

На странице отобразится количество добавляемых на доску задач. При первой выгрузке будут добавлены задачи, которые удовлетворяют сформулированному условию, за исключением:

- недоступных для редактирования тому пользователю, который настраивает автодобавление;
- уже добавленных на доску задач;
- задач, которые попадают под условия автоудаления;
- задач, ранее добавленных на доску автоматически, но удаленных после вручную.

При последующих обновлениях на доску будут добавляться все задачи, подходящие под параметры, указаннные в настройках автодобавления, без учета перечисленных исключений.

## Автоудаление задач {#autodelete}

Действует ограничение на количество задач на доске: их должно быть не больше 2000. Чтобы доска не переполнялась, можно настроить автоматическое удаление задач с доски и, например, удалять все закрытые задачи или задачи, которые были закрыты несколько месяцев назад.

1. На доске нажмите **{{ ui-key.startrek.ui_components_PageAgileBoard_PageAgileBoardHeader.settings }}** → **{{ ui-key.startrek.ui_components_agile_settings_AgileSettingsLayout.issue-updates }}**.
1. В блоке **{{ ui-key.startrek.ui_components_agile_settings_BoardIssueUpdatesSettings.removal-title }}** добавьте все нужные статусы и время нахождения задачи в этом статусе.

Как только условие будет выполнено, задача будет удалена с доски.

## Добавить заметки на доску {#add-note}

Вы можете прикреплять к колонкам доски свои заметки и комментарии, чтобы другим пользователям было проще на ней ориентироваться. Такие заметки не защищены от редактирования — изменить их сможет любой пользователь.

Чтобы прикрепить заметку к колонке:

1. Откройте страницу доски.

1. Вверху колонки нажмите значок ![](../../_assets/tracker/svg/actions.svg) и затем выберите ![](../../_assets/tracker/svg/icon-note.svg)&nbsp;**{{ ui-key.startrek.blocks-desktop_b-agile-board.add-note }}**.

1. Введите текст заметки. При необходимости отформатируйте текст с помощью кнопок на панели.

1. Сохраните изменения.

К каждой колонке можно прикрепить только одну заметку. Она появится в верхней части колонки и будет видна всем пользователям.

## Добавить доску в избранное {#favourites}

Чтобы у вас всегда был быстрый доступ к доске, добавьте ее в избранное. Для этого перейдите на страницу доски и на верхней панели страницы нажмите значок ![](../../_assets/tracker/svg/favourites.svg) справа от названия доски. 

Чтобы найти избранные доски, на панели слева выберите ![](../../_assets/tracker/svg/boards.svg)&nbsp;**{{ ui-key.startrek.ui_components_NavigationBar.boards }}** и отфильтруйте список досок по значению **{{ ui-key.startrek.ui_components_BoardsPanel.favourites }}**.

## Отобразить задачи спринта {#sprint-tasks}

1. Откройте вашу доску.

1. Нажмите **Все задачи на доске**.

1. Выберите спринт, задачи которого нужно отобразить.

## Удалить доску {#delete-board}

Если доска задач больше не нужна, ее может удалить владелец. При этом все задачи, которые были на доске, сохранятся в {{ tracker-name }}. Если удалить доску проекта, задачи сохранятся в [списке задач проекта](project-list.md).

1. Перейдите на доску.

1. В правом верхнем углу нажмите ![](../../_assets/horizontal-ellipsis.svg) и выберите **{{ ui-key.startrek.ui_components_agile_agile-board_AgileBoardActionsMenu.delete-board-menu-item }}**.

1. Подтвердите удаление.
