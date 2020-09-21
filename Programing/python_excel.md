# python 操作 Excel
[TOC]

用Python操作 Excel 2010

```sh
pip install openpyxl
```

## 读
```py
from openpyxl import load_workbook

wb = load_workbook('demo.xlsx')
sheet = wb.get_sheet_by_name("Sheet1")
print sheet.max_row
print sheet.max_column

for i in range(1, sheet.max_row+1):
    for j in range(1, sheet.max_column+1):
        line = sheet.cell(row=i, column=j).value
        print ("%s,%s:%s" %(i, j, line))
```

## 写
```py
from openpyxl import load_workbook
from openpyxl import Workbook
from openpyxl.chart import BarChart, Series, Reference, BarChart3D
from openpyxl.styles import Color, Font, Alignment
from openpyxl.styles.colors import BLUE, RED, GREEN, YELLOW

class Write_excel(object):
    def __init__(self, filename, sheetName):
        self.filename = filename
        self.wb = load_workbook(self.filename)
        self.ws = self.wb.get_sheet_by_name(sheetName)
        # self.ws = self.wb.active

    def write(self, coordinate=None, row=None, column=None, value=None):
        if coordinate:
            self.ws[coordinate].value = value
        elif row and column:
            self.ws.cell(row=row, column=column, value=value)

    def save(self):
        self.wb.save(self.filename)

    def write1(self, coord, value):
        # eg: coord:A1
        #self.ws.cell(coord).value = value
        self.ws[coord].value = value
        self.wb.save(self.filename)

    def write2(self, row, column, value):
        self.ws.cell(row=row, column=column, value=value)
        self.wb.save(self.filename)

    def merge(self, rangstring):
        # eg: rangstring:A1:E1
        self.ws.merge_cells(rangstring)
        self.wb.save(self.filename)

    def cellstyle(self, coord, font, align):
        cell = self.ws[coord]
        cell.font = font
        cell.alignment = align

    def makechart(self, title, pos, width, height, \
        col1, row1, col2, row2, col3, row3, row4):
        ''':param title:图表名
                  pos:图表位置
                  width:图表宽度
                  height:图表高度
        '''
        data = Reference(self.ws, min_col=col1, \
            min_row=row1, max_col=col2, max_row=row2)
        cat = Reference(self.ws, min_col=col3, min_row=row3, max_row=row4)
        chart = BarChart3D()
        chart.title = title
        chart.width = width
        chart.height = height
        chart.add_data(data=data, titles_from_data=True)
        chart.set_categories(cat)
        self.ws.add_chart(chart, pos)
        self.wb.save(self.filename)

def new():
    wb = Workbook()#创建工作簿
    ws = wb.active#激活工作表
    ws1 = wb.create_sheet("Mysheet")#创建mysheet表
    ws.title = "New Title"#表明改为New Title
    ws.sheet_properties.tabColor = "1072BA"#颜色
    ws['A4'] = 4#赋值
    ws['A2'] = 1234#赋值
    d = ws.cell(row=4, column=2, value=10)#赋值
    cell_range = ws['A1':'C2']#选择单元格区域

    wb.save('test.xlsx')#保存

def write():
    wr = Write_excel('test.xlsx')
    wr.write1('B2', 'hello')
    wr.write2(5, 5, 5)
    wr.write(coordinate='A8', value='A8')
    wr.write(row=6, column=6, value=6)
    font = Font(name=u'宋体', size=14, color=RED, bold=True)
    align = Alignment(horizontal='center', vertical='center')
    wr.cellstyle('B2', font, align)
```

