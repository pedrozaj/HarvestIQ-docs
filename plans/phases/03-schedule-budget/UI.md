# Phase 3: UI Pages

Schedule and budget interfaces.

---

## Schedule Tab - Task List View

Default view when clicking Schedule tab.

```
+-----------------------------------------------------------------------------------+
|  [Overview] [Schedule] [Budget] [Documents] [Payments] [Tasks]                    |
|  ================================================================                 |
|                                                                                   |
|  [Task List] [Gantt] [Timeline] [Whiteboard] [Milestones] [Calendar]  [+ Add Task]|
|  ============================================                                     |
|                                                                                   |
|  Phase: [All v]  Status: [All v]  Assignee: [All v]  [________Search______]       |
|                                                                                   |
|  FOUNDATION (3 tasks)                                              [+ Add Task]   |
|  +-----------------------------------------------------------------------+        |
|  | [x] Site preparation         | Joey S.  | Completed | Dec 1-5        |        |
|  | [x] Excavation               | Mike W.  | Completed | Dec 5-8        |        |
|  | [ ] Pour foundation          | Joey S.  | In Progress | Dec 8-12     |        |
|  +-----------------------------------------------------------------------+        |
|                                                                                   |
|  FRAMING (5 tasks)                                                 [+ Add Task]   |
|  +-----------------------------------------------------------------------+        |
|  | [ ] First floor framing      | --       | Not Started | Dec 12-18    |        |
|  | [ ] Second floor framing     | --       | Blocked     | Dec 18-24    |        |
|  |     Blocked by: First floor framing                                  |        |
|  +-----------------------------------------------------------------------+        |
+-----------------------------------------------------------------------------------+
```

### Task Row Elements

| Element | Description |
|---------|-------------|
| Checkbox | Click to complete |
| Name | Task name, click to edit |
| Assignee | User or unassigned |
| Status | Badge with color |
| Dates | Planned date range |
| Actions | Menu on hover |

---

## Schedule Tab - Gantt View

Timeline visualization.

```
+-----------------------------------------------------------------------------------+
|  [Task List] [Gantt] [Milestones] [Calendar]                     [< Dec 2024 >]   |
|  ============================================                                     |
|                                                                                   |
|                        | 1  | 2  | 3  | 4  | 5  | 6  | 7  | 8  | 9  |10 |...     |
|  +---------------------+----+----+----+----+----+----+----+----+----+----+        |
|  | FOUNDATION          |                                                          |
|  | Site preparation    | [======]                                                 |
|  | Excavation          |      [======]                                            |
|  | Pour foundation     |                [==========]                              |
|  +---------------------+                           ◆ Foundation Complete          |
|  | FRAMING             |                                                          |
|  | First floor         |                              [============]              |
|  | Second floor        |                                        ↓ [==========]    |
|  +---------------------+                                                          |
|                                                                                   |
|  Legend: [===] Planned  [###] Actual  ◆ Milestone  ↓ Dependency                   |
+-----------------------------------------------------------------------------------+
```

### Gantt Features

- Phases as grouped rows
- Tasks as horizontal bars
- Milestones as diamonds
- Dependency arrows
- Drag to resize/move (optional)
- Color by status

**Implementation:** Custom-built with React and Tailwind CSS.

---

## Schedule Tab - Timeline View

Horizontally scrollable timeline with tasks grouped by phase.

```
+-----------------------------------------------------------------------------------+
|  [Task List] [Gantt] [Timeline] [Whiteboard] [Milestones] [Calendar]              |
|  ============================================                                     |
|                                                                                   |
|  Legend: [Not Started] [In Progress] [Completed] [Blocked] ◆ Milestone  | Today  |
|                                                                                   |
|  +-----------------------------------------------------------------------------+ |
|  |        Dec 15    Dec 22    Dec 29    Jan 5     Jan 12    Jan 19            | |
|  +-----------------------------------------------------------------------------+ |
|  | Milestones                                                                  | |
|  |    ◆ Foundation     ◆ Framing                                              | |
|  +-----------------------------------------------------------------------------+ |
|  | FOUNDATION (3 tasks)                                          |             | |
|  |    [Site preparation========]                                               | |
|  |        [Excavation===========]                                              | |
|  |              [Pour foundation================]                              | |
|  +-----------------------------------------------------------------------------+ |
|  | FRAMING (5 tasks)                                                           | |
|  |                       [First floor framing==========]                       | |
|  |                                  [Second floor=============]                | |
|  +-----------------------------------------------------------------------------+ |
|                                                   Scroll right to see more →     |
+-----------------------------------------------------------------------------------+
```

### Timeline Features

- Horizontally scrollable (60px per day)
- Tasks stacked vertically per phase
- Phase labels sticky on left
- Week headers at top
- Red "Today" marker line
- Milestones row with diamond markers
- Click task to edit
- Summary footer with stats

---

## Schedule Tab - Whiteboard View

Kanban-style board with draggable sticky notes.

```
+-----------------------------------------------------------------------------------+
|  [Task List] [Gantt] [Timeline] [Whiteboard] [Milestones] [Calendar]              |
|  ============================================                                     |
|                                                                                   |
|  Whiteboard                                        Filter: [All Phases v]         |
|  Drag and drop sticky notes to update task status                                 |
|                                                                                   |
|  +-----------------------------------------------------------------------------+ |
|  | TO DO          | IN PROGRESS     | BLOCKED         | DONE                   | |
|  | (5)            | (3)             | (1)             | (8)                    | |
|  +-----------------------------------------------------------------------------+ |
|  |                |                 |                 |                        | |
|  | +----------+   | +----------+    | +----------+    | +----------+           | |
|  | |[pushpin] |   | |[pushpin] |    | |[pushpin] |    | |[pushpin] |           | |
|  | | Frame    |   | | Pour     |    | | Roof     |    | | Site     |           | |
|  | | windows  |   | | foundation    | | trusses  |    | | prep     |           | |
|  | |          |   | |          |    | |          |    | |          |           | |
|  | | High     |   | | Medium   |    | | Urgent   |    | | Low      |           | |
|  | | Dec 28   |   | | Dec 20   |    | | Jan 5    |    | | Dec 1    |           | |
|  | +----------+   | +----------+    | +----------+    | +----------+           | |
|  |                |                 |                 |                        | |
|  | +----------+   | +----------+    |                 | +----------+           | |
|  | |[pushpin] |   | |[pushpin] |    |                 | |[pushpin] |           | |
|  | | Order    |   | | Install  |    |                 | | Excavate |           | |
|  | | cabinets |   | | conduit  |    |                 | |          |           | |
|  | +----------+   | +----------+    |                 | +----------+           | |
|  +-----------------------------------------------------------------------------+ |
|                                                                                   |
|  Drag notes between columns to update status                                      |
+-----------------------------------------------------------------------------------+
```

### Whiteboard Features

- Drag-and-drop between columns to change status
- Sticky notes colored by phase
- Priority badge on each note
- Phase filter dropdown
- Push-pin visual on each note
- Cork board aesthetic
- Click to edit task details

---

## Schedule Tab - Milestones View

```
+-----------------------------------------------------------------------------------+
|  [Task List] [Gantt] [Milestones] [Calendar]                    [+ Add Milestone] |
|  ============================================                                     |
|                                                                                   |
|  UPCOMING                                                                         |
|  +-----------------------------------------------------------------------+        |
|  | ◆ Foundation Complete      | Foundation | Dec 15, 2024  | 5 days     |        |
|  | ◆ Framing Inspection       | Framing    | Dec 28, 2024  | 18 days    |        |
|  +-----------------------------------------------------------------------+        |
|                                                                                   |
|  ACHIEVED                                                                         |
|  +-----------------------------------------------------------------------+        |
|  | ✓ Permits Approved         | --         | Dec 1, 2024   | On time    |        |
|  +-----------------------------------------------------------------------+        |
|                                                                                   |
|  MISSED                                                                           |
|  +-----------------------------------------------------------------------+        |
|  | ✗ Site Survey              | --         | Nov 28        | 2 days late|        |
|  +-----------------------------------------------------------------------+        |
+-----------------------------------------------------------------------------------+
```

---

## Schedule Tab - Calendar View

```
+-----------------------------------------------------------------------------------+
|  [Task List] [Gantt] [Milestones] [Calendar]                          [Week v]    |
|  ============================================                                     |
|                                                                                   |
|  < December 2024 >                                                                |
|  +-----------------------------------------------------------------------+        |
|  | Sun     | Mon     | Tue     | Wed     | Thu     | Fri     | Sat     |        |
|  +-----------------------------------------------------------------------+        |
|  | 1       | 2       | 3       | 4       | 5       | 6       | 7       |        |
|  | [Permits| [Site   |         |         | [Excav  |         |         |        |
|  |  Appr.] |  Prep]  |         |         |  -ation]|         |         |        |
|  +-----------------------------------------------------------------------+        |
|  | 8       | 9       | 10      | 11      | 12      | 13      | 14      |        |
|  | [Pour   |         |         |         | [First  |         |         |        |
|  |  Found.]|         |         |         |  Floor] |         |         |        |
|  +-----------------------------------------------------------------------+        |
+-----------------------------------------------------------------------------------+
```

**Library recommendation:** react-big-calendar

---

## Budget Tab

```
+-----------------------------------------------------------------------------------+
|  [Overview] [Schedule] [Budget] [Documents] [Payments] [Tasks]                    |
|  ================================================================                 |
|                                                                                   |
|  BUDGET OVERVIEW                                                                  |
|  +---------------------------+  +---------------------------+                     |
|  | TOTAL BUDGET              |  | BY CATEGORY (pie chart)   |                     |
|  | $500,000 estimated        |  |   Framing: 25%            |                     |
|  | $450,000 actual           |  |   Electrical: 18%         |                     |
|  | [================---] 90% |  |   Plumbing: 15%           |                     |
|  | $50,000 under budget      |  |   ...                     |                     |
|  +---------------------------+  +---------------------------+                     |
|                                                                                   |
|  BUDGET ITEMS                                              [+ Add Budget Item]    |
|  Category: [All v]  Phase: [All v]  [________Search______]                        |
|                                                                                   |
|  +-----------------------------------------------------------------------+        |
|  | Name              | Category    | Phase      | Est.     | Actual   |V|        |
|  +-----------------------------------------------------------------------+        |
|  | Main panels       | Electrical  | Rough-in   | $15,000  | $14,200  |+|        |
|  | Wiring labor      | Electrical  | Rough-in   | $8,000   | $9,500   |-|        |
|  | Fixtures          | Electrical  | Finish     | $5,000   | $4,800   |+|        |
|  +-----------------------------------------------------------------------+        |
|                                                                                   |
|  V = Variance: + under budget, - over budget                                      |
+-----------------------------------------------------------------------------------+
```

### Budget Item Modal

```
+------------------------------------------------------------------+
|  Add Budget Item                                            [X]   |
+------------------------------------------------------------------+
|                                                                   |
|  Category *                                                       |
|  [Electrical                                            v]        |
|                                                                   |
|  Phase (optional)                                                 |
|  [Rough-in                                              v]        |
|                                                                   |
|  Name *                                                           |
|  [Main electrical panel                                  ]        |
|                                                                   |
|  Description                                                      |
|  [200 amp service upgrade                                ]        |
|                                                                   |
|  Estimated Amount *             Actual Amount                     |
|  [$  15,000.00    ]             [$  0.00        ]                 |
|                                                                   |
|                                     [Cancel]  [Add Item]          |
+------------------------------------------------------------------+
```

---

## Create Task Modal

```
+------------------------------------------------------------------+
|  New Task                                                   [X]   |
+------------------------------------------------------------------+
|                                                                   |
|  Phase *                                                          |
|  [Foundation                                            v]        |
|                                                                   |
|  Task Name *                                                      |
|  [_______________________________________________________]        |
|                                                                   |
|  Description                                                      |
|  [_______________________________________________________]        |
|                                                                   |
|  Assigned To                     Priority                         |
|  [Joey Smith                v]   [Medium                    v]    |
|                                                                   |
|  Planned Start              Planned End                           |
|  [____/__/____]             [____/__/____]                        |
|                                                                   |
|  Estimated Hours                                                  |
|  [________]                                                       |
|                                                                   |
|  Dependencies (optional)                                          |
|  [Select tasks this depends on...                       v]        |
|  [x] Site preparation                                             |
|  [x] Excavation                                                   |
|                                                                   |
|                                       [Cancel]  [Create Task]     |
+------------------------------------------------------------------+
```

---

## Components to Build

| Component | Purpose |
|-----------|---------|
| TaskList | Grouped task display |
| TaskRow | Individual task with actions |
| TaskForm | Create/edit task modal |
| PhaseHeader | Collapsible phase section |
| GanttChart | Gantt chart visualization |
| TimelineChart | Horizontally scrollable timeline |
| WhiteboardView | Kanban-style sticky note board |
| CalendarView | Calendar display |
| MilestoneList | Milestone grouping |
| MilestoneForm | Create/edit milestone |
| BudgetOverview | Summary cards and chart |
| BudgetTable | Budget items list |
| BudgetItemForm | Create/edit budget item |
| CategorySelect | Category dropdown |
| DependencySelect | Multi-select for dependencies |
