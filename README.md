# odoo16建設環境 

在使用odoo之前，我們需要把環境先準備好，這篇文章主要使用Windows11來做為示範，而odoo的安裝方式有很多種，
在這邊我們使用Source的方式來進行安裝。
### ○ Python
    安裝 python3.7 https://www.python.org/downloads/
目前odoo使用python3.7x版本是相對穩定的，筆者建議讀者安裝這個版本，並且輸入以下指令來更新pip工具確定在最新的版本。
```CMD
python -m pip install --upgrade pip
```
### ○ Git
    安裝 git  https://git-scm.com/downloads
在安裝Source odoo16以前，我們要先將Git本版控制工具給下載好。
### ○ Postgresql
    安裝 PostgreSQL https://www.postgresql.org/download/
這是odoo默認的關聯式資料庫，也一併需要安裝。
### ○ Pycharm
    安裝 pycharm Community https://www.jetbrains.com/pycharm/download/?section=windows
在開發時，筆者推薦使用Pycharm，作為IDE來使用。
### ○ Source Odoo16
    安裝 odoo16 git clone https://github.com/odoo/odoo.git
打開CMD，複製上面這段指令，即可下載odoo16。
# 開啟odoo16
在開始使用odoo16之前，我們必須先透過腳本requirements.txt來進行python套件的安裝，並且在venv的環境中安裝，在這邊我們使用Pycharm
內建設定來使用後面會詳細介紹，在安裝途中會有一些bug，你可以開啟requirements.txt先將該問題套件字段刪除，之後再進行手動安裝。
```CMD
pip install -r requirements.txt
```
再來我們可以套過安裝PostgreSQL得到的工具[SQL Shell(psql)]，來創建專用DB一下為示範。
```CMD
Server[localhost]: localhost
Database [postgres]: postgres
Port[5432]: 5432
Username [postgres]: postgres
Password for user postgres: --安裝當中所設定的密碼--
```
登入稱成功後，你會看到下面這段文字
```CMD
psql(--你安裝PostgreSQL的版本--)
Type "help" for help.

postgres=#
```
再來我們來創建DATABASE跟USER
```CMD
postgres=# CREATE DATABASE odoo16;
```
```CMD
postgres=# CREATE USER odoo16 WITH PASSWORD 'odoo16';
```
完成之後，我們就可以開啟我們的odoo16了。
```CMD
python odoo-bin -d odoo16 -r odoo16 -w odoo16 -i base
```
在執行這段指令的時候記得，一定要在你odoo16的資料夾下。
### ○ 常用指令
一般我們使用odoo的時候會喜歡將依些指令檔設定在odoo16的文件中的config文件中，內容在下圖中可以看到。
```CMD
位置: ~/odoo16/config/odoo.conf
```
.圖片odoo.conf
```CMD
python odoo-bin -c [自訂config檔路徑]
```
這個就是使用該odoo.conf的指令。
```CMD
python odoo-bin --dev=all
```
一般在開發中，我們再啟動odoo的指令中加上`--dev=all`之後，我們可以即時的在開發途中進行更新，不需要再重啟整個odoo。
```CMD
python odoo-bin -d odoo16 -r odoo16 -w odoo16 -i base --without-demo=all
```
在最開始我們啟動odoo的時候，會將demo的資料全部匯進去，如果不希望匯入可以加上這段`--without-demo=all`
### ○ pycharm設定環境
```CMD
設定Pycharm venv的圖
```
```CMD
設定Pycharm compiler的圖
```
### 自動創建addons指令
```CMD
python odoo-bin scaffold [addons名稱] [資料夾像要設定的路徑]
```
上面的指令會生成下面資料夾與檔案
```CMD
addons名稱/
├── controllers
│   ├── controllers.py
│   └── __init__.py
├── demo
│   └── demo.xml
├── __init__.py
├── __manifest__.py
├── models
│   ├── __init__.py
│   └── models.py
├── security
│   └── ir.model.access.csv
└── views
    ├── templates.xml
    └── views.xml
```
通常`[資料夾像要設定的路徑]`我們會設定在odoo16一樣的根目錄，以防更新的時候去影響。
### ○ `__manifest__.py`
在`__manifest__.py`檔案中，會長這個樣子
```CMD
{
    'name': "My library",
    'summary': "輕鬆圖書館",
    'description': """
     Manage Library
     ==============
     Description related to library.
     """,
    'author': "Dokuro",
    'website': "https://https://eoffice.alltop.com.tw/eOffice2007/index.php",
    'category': 'Uncategorized',
    'version': '16.0.1',
    'depends': ['base'],
    'data': [
    'security/groups.xml',
    'security/ir.model.access.csv',
    'views/library_book.xml'
    ],
    'installable':True,
    'auto_install':False,
    ],
}
```
* name: 代表這個addons的名稱。
* summary: 簡單的介紹模組。
* description: 詳細的介紹模組。
* author: 代表addons的作者。
* website: 模組相關的網站。
* category: 默認 'Uncategorized'。
* depends: 跟繼承有關係後面會詳細說明。
* data: 是用來我們想要呈現的view，通常建議從有權限的開始添加。

# 外部ID XML ID
### 在odoo中我們常常會使用到XML ID主要都在繼承上使用比如說:
我們想要繼承某個view的時候假設我們的資料夾在`module/view/test.xml`下我們想創建一個view時
* `<record id=testing model='ir.ui.view'>`在這當中我們不僅創建了`test.testing`的XML ID同時我們也呼叫了`ir.ui.view`
這段外部ID

簡單說XML ID就是DB給予record的別名，並且不可以重複。
## 再來我們就要開始進入odoo-MVC架構中的開發介紹
常用框架MVC在odoo中，M代表Model負責欄位邏輯，V代表view負責user_interface也就是UI的呈現，C代表Cotroller
負責路由，那我們就開始認識他們吧。
# Model
### ○ (model.Models) - 最常使用的model類型
`my_module/__init__.py`
在開始controller設定之前，我們必須先將我們models給import進去
```CMD
from . import models
```
`my_module/models/__init__.py`
在開始controller設定之前，我們必須先將我們models.py給import進去
```CMD
from . import library_model
```
### ○ 種類和使用
```CMD
from odoo import models,fielsd,api

class Library(models.Model):
    _name = 'library.book'
    _description = 'library.book'
    _inherit='mail'
    _order='sequence,name'
```
* name: model的名稱。
* description: model檢的的描述。
* inherit:繼承自這個model的屬性。
* order: 主要是排序，跟SQL中的`order by`是類似的。

※當我們想要呼叫Library當中的欄位資料時，我們可以使用`self.env['library.book']`
```CMD
    fields.Char() - 常用屬性有string(代表欄位的名子)跟size(可以設定最長長度)
    fields.Text() - 常用屬性為string(代表欄位的名子)
    fields.Integer() - 常用屬性為string(代表欄位的名子)
    fields.Float() - 常用屬性有string(代表欄位的名子)跟digits(通常會是一個tuple並且分別代表('總位數','小數精度'))
    fields.Html() - 常用屬性為string(代表欄位的名子)
    fields.Date() - 常用屬性string(代表欄位的名子)或是default=lambda = self:fields.Date.now()
    fields.Datetime() - 常用屬性string(代表欄位的名子)或是default=lambda = self:fields.Datetime.now()
    fields.Binary() - 常用屬性為string(代表欄位的名子)
    fields.Selection() - 常用屬性有items(會是一個[list])跟string(代表欄位的名子)。
```
※Date()與Datetime還有這些類型:
```CMD
fields.Date.to_date(string_value)
fields.Date.to_string(date_value)
fields.Date.today()
fields.Date.context_today(record, timestamp)
---------------------------------------------
fields.Datetime.to_date(string_value)
fields.Datetime.to_string(datetime_value)
fields.Datetime.today()
fields.Datetime.context_today(record, timestamp)
```

上面這些都是常用的`欄位`常用的屬性:
* string: 欄位名稱
* required: 必填欄位
* help: 提示訊息
* compute : 可以對欄位的資料做及時的計算，不會回存到table中但可以透過store=True來進行儲存。
* default : 設定預設值。
* translate : 通常在Char、Text、Html中使用，主要使用在翻譯。
* readonly : 預設為False，只可以讀不能編輯
* index : 設定為True之後，會優先被查找減輕DB負載。
* copy : 設定該欄位能不能被複製。
* groups : 設定只有該群組可以看到欄位。
* states : 通常長這樣{'done':['readonly',True]}，用這樣的方式來設定欄位只能被讀。還可以設定required、invisible。
* deprecated : 只要將其設定為True，會在日誌中記錄警告訊息。
* active : 如果降他設定為False，當前端在查詢時將會被忽略。
* sequence : 必須在_order中引入，可以手動定義record順序。

### ○ @api
常用的@api有以下這些:
* @api.model  - 通常會運用在修改既有CRUD函式時會呼叫它，而在odoo16它已經取代了@api.one與@api.multi的功能了。
* @api.constrains(" ") - 主要的功能是在約束fields，與`_sql_constraints`是擁有相同的功能。
* @api.depends(" ")    - 通常與compute一起使用，只能在view上做操作，操作過的record不會儲存在table中。
* @api.onchange        - 可以對record資料做計算，但只能在同一model中做使用。

大多數情況，使用@api都是為了DB效能而使用，當你使用方法在抓取recordset的時候你沒有使用@api
,odoo會預設為@api.model來實行。


內建的CRUD:
* read([])
* create({})
* write({})
* unlink()
* browse(int)
* search()
當我們想使用其他model的recordset的時候，我們可以這樣做:
```CMD
def other_recordset(self):
    all_members=self.env['library.member'].search([])
    print("All members:",all_members)
    return True
```
或是我們想要創建一個新record:
```CMD
record=self.evn['library.book.category'].create(parent_category_val)
```
還可以更新record:
```CMD
def update_record(self):
    self.ensure_one()
    self.date_release=fields.Date.today()
------------------------------------------
def update_record(self):
    self.ensure_one()
    self.update({'date_release':'fields.Date.today()'})
```
我們也可以透過write({})來對record的操作
```CMD
record.write({'child_ids':[0,0,child]})
record.write({'child_ids':[1,child_index,update_child]})
```
這邊詳細介紹這些`( , , )`內所代標的意思
```CMD
(0,0,{a:b})   - 創建record,並且將{a:b}關聯
(1,id,{a:b})  - 更新id值為id的record,並且將{a:b}關聯
(2,id,)       - 刪除id值為id的record
(3,id,)       - unlink id值為id的record
(4,id,)       - link id值為id的record
(5,0,0)       - 刪除所有的 link record
(6,0,[ids])      - 替換現有的 link record
```

### ○ 報錯
```CMD
from odoo.exceptions import UserError
from odoo.tools.translate import _
```
首先我們要import UserError跟_
```CMD
@api.model
def change_state(self, new_state):
    for book in self:
        if book.is_allowed_transition(book.state, new_state):
            book.state = new_state
        else:
            msg = _('Moving from %s to %s is not allowed') % (book.state, new_state)
            raise UserError(msg)
```
這邊這段程式碼的用意，主要確認借書狀態是否可以更新，如果不行將回傳Error給使用者，其中`_`
在未來翻譯多國語言的時候可以用到。
```CMD
ValidationError -沒有滿足constrains條件報錯
AccessError -用戶不是群組內的使用者，不得訪問
RedirectError -報出Error後，指向指定View當中
```
這些是其他常用報錯的字段
### ○ Filter record
如果我們想要將特定的record找出來我們可以透過以下方法:
```CMD
@api.model
def multiple_authors(self,all_book):
    def predicate(book):
        if len(book.author_ids)>1:
            return True
        return all_book.filter(predicate)
```
```CMD
@api.model
def get_author_names(self,books):
    return book.mapped('author_ids.name')
```
或是排序record:
```CMD
@api.mode
def sort_book_by_date(self,books):
    return books.sorted(key='release_date')
```
### ○ sudo()
當我們遇到需要特定全線record時，想將資料呈現給不再此群組中的使用者看，可以使用:
```CMD
def book_rent(self):
    self.ensure_one() #這段在使用sudo()時，非常重要必定要搭配使用
    if self.state !='avaliable':
        raise UserError(_('book is not available for renting!!'))
        rent_as_superuser=self.env['library.book.rent'].sudo()
        rent_as_superuser.create({'book_id':self.id,
                                  'borrower_id':self.env.user.partner_id.id,})
```
另外還有兩種model:
* model.AbstractModel - 通常會跟report一起使用，並且資料不會被儲存。
* model.transientModel - 通常會用在引導用戶使用，並且是臨時儲存的，在一定的時間內就會被系統刪除舊的數據。
# View
記得我們如果想要看的畫面的話，一定要在`__manifest__.py`中加入:
```CMD
    'data': [
                .
                .
                .
    'views/library_book.xml'
    ],
```
### ○ 種類
`<Form>`表單
#### 表單跟列表是最常用的欄位表示方式，通常在表單當中`<header>`跟`<footer>`會放`<button>`另外`<header>`
還會放進度條，而<sheet>通常會跟<group>搭配來放上欄位:
```CMD
<form>
    <header>
        .
        .
        .
    </header>
    <sheet>
        <group>
            <field name="name"/>
            <field name="function"/>
        </group>
    </sheet>
    <footer>
        .
        .
        .
    </footer>
</form>
```
`<Tree>`列表比較簡單，通常只會單純的在欄位做顯示:
```CMD
<tree>
    <field name="name"/>
    <field name="email"/>
    <field name="phone"/>
</tree>
```
`<Kanban>`可以做為按鈕來顯示，並且樣式更多元，可以搭配CSS來使用:
```CMD
<kanban>
    <field name="name" />
    <field name="supplier" />
    <field name="customer" />
    <templates>
        <t t-name="kanban-box">
            <div class="oe_kanban_card">
                <a type="open">
                    <strong>
                        <field name="name" />
                    </strong>
                </a>
                <t t-if="record.supplier_rank.raw_value or record.customer_rank.raw_value">
                    is
                    <t t-if="record.customer_rank.raw_value">
                        a customer
                    </t>
                    <t t-if="record.supplier_rank.raw_value and record.customer_rank.raw_value">
                        and
                    </t>
                    <t t-if="record.supplier_rank.raw_value">
                        a supplier
                    </t>
                </t>
            </div>
        </t>
    </templates>
</kanban>
```
`<Notebook>`使用標籤分頁:
```CMD
<notebook>
  <page string="Tab 1">
    <field name="field1"/>
    <field name="field2"/>
    <field name="field3"/>
  </page>
  <page string="Tab 2">
    <field name="field4"/>
    <field name="field5"/>
    <field name="field6"/>
  </page>
</notebook>
```
`<Calendar>`日曆只需要加上欄位後，就可以使用:
```CMD
<calendar date_start="date_assign" date_stop="date_end" color="project_id">
    <field name="name"/>
    <field name="user_id"/>
</calendar>
```
`<Pivot>`透視表示圖只需要加上欄位後，就可以使用:
```CMD
<pivot>
    <field name="user_id" type="row"/>
    <field name="project_id" type="col"/>
    <field name="stage_id" type="col"/>
</pivot>
```
`<Graph>`柱狀、折線、圓餅圖只需要加上欄位後，就可以使用:
```CMD
<graph type="bar">
    <field name="user_id" type="row"/>
    <field name="stage_id" type="col"/>
</graph>
```
`<Search>`搜尋視圖通常會搭配`<field name="category_id" filter_domain="[]"/>`來做搜尋，當然也可以直接使用
`<filer>`或是`<group expand="0" string="Group By">`
```CMD
<search>
        <field name="name"/>
        <field name="category_id" filter_domain="[('category_id', 'child_of', self)]"/>
        <field name="bank_ids" widget="many2one"/>
        <filter name="suppliers" string="Suppliers" domain="[('supplier_rank', '>', 0)]"/>
        <group expand="0" string="Group By">
            <filter string="Country" name="country" context="{'group_by':'country_id'}"/>
        </group>
</search>
```
`<Button>`按鈕:
#### 當`<butto type='action'>`時，觸發之後會彈出`base.action_partner_category_form`:
```CMD
<button type='action' name="%(base.action_partner_category_form)d"> string="open category"/>
```
#### 當`<butto type='object'>`時，觸發之後會調用model中的方法`open_commericail_entity`:
```CMD
<button type="object" name="open_commercial_entity" string="Open commercial partner" class="btn-primary" />
```
### ○ Action
action通常控制，頁面呈現所需要的元素，通常我們想使用表單或是列表這些view的時候我們也會在這邊先宣告`view_mode="tree,form"`。
```CMD
<record id='action_all_customers' model='ir.actions.act_window'>
  <field name="name">All customers</field>
  <field name="res_model">res.partner</field>
  <field name="view_mode">tree,form</field>
  <field name="domain">[('customer', '=', True)]</field>
  <field name="context">{'default_customer': True}</field>
  <field name="limit">20</field>
</record>
------------------------------------------------------------------
<act_window id="action_all_customers"
  name="All customers"
  res_model="res.partner"
  view_mode="tree,form"
  limit ="20"
  domain="[('customer', '=', True)]"
  context="{'default_customer': True}" /> 
```
不管是使用`<record id='action_all_customers' model='ir.actions.act_window'>>`或是`<act_window>`
呈現出來的效果都是一樣的。
### ○ Menuitem
menuitem負責呈現目錄，讓使用者可以在UI見面更直覺的使用odoo所提供的功能。
```CMD
root menu
---------------------------------------------------------------------------------------------------------
<menuitem id="menu_custom_top_level" name="My App menu" web_icon="my_module,static/description/icon.png"/>

菜單 menu
-------------------------------------------------------------------------------------------------------------
<menuitem id="menu_all_customers" parent="menu_custom_top_level" action="action_all_customers" sequence="10"/>
```
當我們想要透過menu來打開tree、form或是search的時候我們可以這麼做:
```CMD
Tree
<record id="view_all_customers_tree" model="ir.ui.view">
  <field name="name">All customers</field>
  <field name="model">res.partner</field>
  <field name="arch" type="xml">
    <tree>
      <field name="name" />
    </tree>
  </field>
 </record>
 ---------------------------------------------------------
 Form
 <record id="view_all_customers_form" model="ir.ui.view">
  <field name="name">All customers</field>
  <field name="model">res.partner</field>
  <field name="arch" type="xml">
    <form>
      <group>
        <field name="name" />
      </group>
    </form>
  </field>
 </record>
 ---------------------------------------------------------
 Search
 <record id="search_all_customers" model="ir.ui.view">
    <field name="model">res.partner</field>
    <field name="arch" type="xml">
        <search>
            <field name="name"/>
            <field name="category_id" filter_domain="[('category_id', 'child_of', self)]"/>
            <field name="bank_ids" widget="many2one"/>
            <filter name="suppliers" string="Suppliers" domain="[('supplier_rank', '>', 0)]"/>
            <group expand="0" string="Group By">
                <filter string="Country" name="country" context="{'group_by':'country_id'}"/>
            </group>
        </search>
    </field>
</record>
 ```
### ○ Parameter參數
在`<action>`當中
1. id         - 設定XML ID
2. name       - 這個action的名子 
3. res_model  - 填入要使用model的_name
4. view_mode  - 列出要使用的view如:form或是tree
5. target     - 用來呈現動作如果填入new，則會彈出一個視窗
6. context    - 用來設定欄位預設值，或是激活filter
7. domain     - 可以透過所設定的值，來讓顯示我們想讓使用者看到的資料內容
8. limit      - 設定所顯示的record數量。
#### 
在`<menuitem>`當中
1. id       - 設定XML ID
2. name     - menu的呈現的名子
3. parent   - 來自哪個root_menu，填入的值會是XML ID
4. action   - 使用哪一個action，填入的值會是XML ID
5. sequence - 設定數字來指定在root menu下的順序
#### 
在`<record>`當中
1. id     - 設定XML ID
2. model  - 繼承來自哪個model
#### 
在`<record>`當中的`<field name=''>`
1. name  -設定顯示名稱
2. model -來自哪個model，填入該XML ID
3. arch  -通常搭配tree、form這些view，後最還會搭配`type='xml'`
4. context  - 用來設定欄位預設值，或是激活filter
5. domain  - 可以透過所設定的值，來讓顯示我們想讓使用者看到的資料內容
6. optional - 設定可以是show或是hide
7. widget - 小工具，可以是text/teatarea/seleciton/date/datetiome/boolean.

####
在`<field name='category_id'>`
* attrs="{'readonly':('status','=','done')}" - 在status為done時readonly=True
* groups="base.group_on_one"  - 只有base.group_on_one群組可以看到此欄位
##### 另外在attrs中也常用`oe_read_only` `oe_edit_only` `oe_inline`
##### 依照順序分別表示 '只能讀'、'只能編輯'、'只能在同一行'

### `<field name="inherit_id">`View的繼承
`<form>`的繼承
#### 下面這對程式碼繼承至`base.view_partner_form`在website之後添加`<field name="write_date"/>`欄位
```CMD
<record id="view_partner_form" model="ir.ui.view">
    <field name="model">res.partner</field>
    <field name="inherit_id" ref="base.view_partner_form"/>
    <field name="arch" type="xml">
        <field name="website" position="after">
            <field name="write_date"/>
        </field>
    </field>
</record>
```
`<tree>`的繼承
#### 下面這對程式碼繼承至`base.view_partner_tree`在email之後添加`<field name="phone" position="move"/>`欄位
```CMD
<record id="view_partner_tree" model="ir.ui.view">
    <field name="model">res.partner</field>
    <field name="inherit_id" ref="base.view_partner_tree"/>
    <field name="arch" type="xml">
        <field name="email" position="after">
            <field name="phone" position="move"/>
        </field>
    </field>
</record>
```
`<Search>`的繼承
#### 下面這對程式碼繼承至`base.view_res_partner_filter`添加到收尋選項最下層`<field name="mobile"/>`欄位
```CMD
<record id="view_res_partner_filter" model="ir.ui.view">
    <field name="model">res.partner</field>
    <field name="inherit_id" ref="base.view_res_partner_filter"/>
    <field name="arch" type="xml">
        <xpath expr="." position="inside">
            <field name="mobile"/>
        </xpath>
    </field>
</record>
```
※`<record id="指定的view">`繼承所指定的view不能更改
## `<tree>`可以加上文字屬性跟顏色
```CMD
<tree decoration-bf="customer_rank &gt;0"
        decoration-danger="customer_rank &gt;0 and supplier_rank &gt;0"
        decoration-warning ="supplier_rank &gt; 0">
    <field name="name"/>
    <field name="user_id" widget="many2one_avatar_user"/>
    <field name="state_id" optional="hide"/>
    <field name="country_id" optional="show"/>
    <field name="customer_rank" invisible="1"/>
    <field name="supplier_rank" invisible="1"/>
</tree>
```
※`decoration-bf`為粗體、`decoration-danger`顯示紅色、`decoration-warning`顯示黃色
## `<Search>`通常使用三種寫法
會在搜尋欄位出現name、category_id、bank_ids來做為搜尋的依據:
```CMD
<search>
    <field name="name"/>
    <field name='category_id" filter_domain="[('category_id','child_id','self')]"/>
    <field name="bank_ids" widget="many2one"/>
</search>
```
會在Filters裡面作用，以Suppliers作為過濾的依據:
```CMD
<search>
    <filter name="suppliers" string="Suppliers" domain="[('supplier_rank', '>', 0)]"/>
</search>
```
會在Group By裡面作用，以Conutry作為分組的依據:
```CMD
<search>
    <group expand="0" string="Group By">
        <filter string="Country" name="country" context="{'group_by':'country_id'}"/>
    </group>
</search>
```
#### ※可以加入這段來使的畫面出現左邊分類工具欄
```CMD
<searchpanel>
    <field name="user_id" icon="fa fa-users"/>
    <field name="category_id" icon="fa fa-list" select="multi"/>
</searchpanel>
```
`form_view_ref`與`tree_view_ref`可以指定開啟的view
```CMD
<field name="context">
    {'tree_view_ref':'my_module.tree_all_contants'}
</field>
```

# Controller
`my_module/__init__.py`
在開始controller設定之前，我們必須先將我們controllers給import進去
```CMD
from . import contorllers
```
`my_module/controllers/__init__.py`
在開始controller設定之前，我們必須先將我們library_controller.py給import進去
```CMD
from . import library_contorller
```
### ○ `@http.route`
```CMD
from odoo import http
from odoo.http import request

class library_contorller(http.Controller):
  @http.route('/my_library/books',method=['GET'], type='http', auth='none')
  def books(self):
    books = request.env['library.book'].sudo().search([])
    html_result = '<html><body><ul>'
    for book in books:
      html_result += "<li> %s </li>" % book.name
    html_result += '</ul></body></html>'
    return html_result

```
在`@route('/my_library/books', type='http', auth='none')`當中`'/my_library/books'`就是我們的網址後綴，還有我們使用了`type='http'`，透過`request.env[]`將`library.book`中
所有recordset給抓出來並透過for來將他們列出來 ; `auth='none'`代表任何人都可以訪問`.../my_library/books`這個網址。
###
或者我們也可以指定template來顯示:
```CMD
from odoo import http

class LibraryController(http.Controller):
    @http.route('/my_library/books', auth='user')
    def list_books(self, **kwargs):
        book_model = http.request.env['library.book']
        books = book_model.sudo().search([])
        return http.request.render('library.library_template', {'books': books})
```
我們透過render來將objs的值，回傳給library_template當中對應的'objs'，並且這次我們將指定只有user群組可以看到畫面
`auth='user'`。
### ○ api
我們也可以透過json來製作api
```CMD
import json
from odoo import http, tools

class BookController(http.Controller):
    @http.route('/book', methods=['POST'], type='json', auth='public', csrf=False)
    def create_book(self, **kwargs):
        data = http.request.jsonrequest
        book_name = data.get('name')
        author = data.get('author')
        result = {'author': author}
        response = json.dumps(result, default=tools.json_default)
        return http.Response(response, status=200, headers=[('Content-Type', 'application/json')])

```
```CMD
from odoo import http

@http.router('/book',metho=['POST'],type='json',auth='*',csrf=False)
     def create_book(self, **kwargs):
        data = http.request.jsonrequest
        book_name = data.get('name')
        author = data.get('author')
        result = {'author': author}
        response = json.dumps(result, default=data_utils.json_default)
        return response
```
### ○ `website=True`
當我們在@http.router中加入`website=True`之後，我們將會直接套用odoo的網頁主題:
#### 首先我們要在__manifest__.py中加入website:
```CMD
'depends': [．．．, 'website'],
```
之後我們在先前的程式碼加入`website=True`:
```CMD
from odoo import http

class LibraryController(http.Controller):
    @http.route('/my_library/books', auth='user', website=True)
    def list_books(self, **kwargs):
        book_model = http.request.env['library.book']
        books = book_model.sudo().search([])
        return http.request.render('library.library_template', {'books': books})
```
# Security
我們在創建權限的時候，一定要在`__manifest__.py`中加入，而且必須以權限大的優先排列:
```CMD
'data': [
    'security/groups.xml',
    'security/ir.model.access.csv',
    'views/library_book.xml'
    ],
```
### ○ 權限分類
##### 通常創建權限3個面向
1. 對象級別 - 對DB的訪問權，可以對recordset進行CRUD的操作。
2. 視圖級別 - 不同用戶組所能看到的畫面呈現與細項。
3. 字段級別 - 對字段的訪問權限，而字段指的就是DB中Table裡的資料。
### ○ create group
收仙我們來創建群組，我們要先在`__manifest__.py`中寫入:
```CMD
'catetgory':'library',
```
※這個設定使的odoo會生成`module_category_library`在base下。

再來新增我們groups.xml的檔案:
```CMD
'data': [
    'security/groups.xml',
                ．
                ．
    ],
```
開始創建 User 與 Librarian 的群組:
```CMD
<odoo>
    <record id='library_user' model='res.gruops'>
        <field name="name">User</field>
        <field name="category_id" ref="base.module_category_library"/>
        <field name="implied_ids" eval="[(4,ref("base.group_user"))]"/>
    </record>
    
    <record id='library_librarian' model='res.gruops'>
        <field name="name">Librarian</field>
        <field name="category_id" ref="base.module_category_library"/>
        <field name="implied_ids" eval="[(4,ref("library_user"))]"   <!--─╥──(user+user_admin的權限)  -->
        <field name="users" eval="[(4,ref("base.user_admin"))]"/> <!--────╜    -->
    </record>
</odoo>
```
### ○ model access
通常檔案的名稱就叫做ir.model.access.csv
```CMD
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
acl_book,library.book_default,model_library_book,group_library_user,1,0,0,0
acl_book_librarian,library.book_librarian,model_library_book,group_library_librarian,1,1,1,1
```
### ○ 權限(0,0,{})~(6,0,[])
```CMD
(0,0,{a:b})   - 創建record,並且將{a:b}關聯
(1,id,{a:b})  - 更新id值為id的record,並且將{a:b}關聯
(2,id,)       - 刪除id值為id的record
(3,id,)       - unlink id值為id的record
(4,id,)       - link id值為id的record
(5,0,0)       - 刪除所有的 link record
(6,0,[ids])      - 替換現有的 link record
```

範例:假設我們想限制model字段的訪問:
```CMD
is_public = fields.Boolean(group="my_library.group_library_user")
private_notes = fields.Text(group="my_library.group_library_librarian")
```

### ○ `ir.rule`
當我們想要限制record的訪問，我們會創建`Security/library_security.xml`，比如說user可以看自己的書
，librarian可以看到所有的書
```CMD
<odoo noupdate="1">
    <record id="library_book_user_rule" model="ir.rule">
        <field name="name">Library: See only own books</field>
        <field name="model_id" ref="model_library_book"/>
        <field name="groups" eval="[(4, ref('my_library.group_library_user'))]"/>
        <field name="domain_force">[('is_public','=',True)]</field>
    </record>
    
    <record id="library_book_all_rule" model="ir.rule">
        <field name="name">Library: See all books</field>
        <field name="model_id" ref="model_library_book"/>
        <field name="groups" eval="[(4, ref('my_library.group_library_librarian'))]"/>
        <field name="domain_force">[(1,'=',1)]</field>
    </record>
</odoo>
```
### ○ superuser -sudo()
當我們在畫面中，不是user權限內可以使用的功能時，我們可以使用sudo()與ensure_one():
```CMD
def report_missing_book(self):
    self.ensure_one()
    message = "Book is missing (Reported by: %s)" % self.env.user.name
    self.sudo().write({
        'report_missing': message
    })
```
※如果我們想要隱藏menu或是header的話，我們也可以使用groups:
```CMD
<menuitem id='library_book_category_menu' name='book category'
          parent='library_book_category_form' 
          groups='my_library.group_library_librarian'/>
```
```CMD
    .
    <form>
        <header groups="my_library.group_library_user"．．．．．>
            .
            .
        </header>
        .
        .
        .
    </form>
    .
```
※我們也可以透過XML中的`<delete>`、`<function>`來做操作:
我們可以透過`<delete>`來進行刪除:
```CMD
<delete model=ir.model.access search='[('id','=',ref(library.library_book_user_rule))]'>
```
或是透過`<function>`來觸發函式:
```CMD
<function model="library.book" name="book_missing" eval="[ref('book_id'),ref('book_author')]">
```
# 使用原生SQL語法
### ○ RAW SQL 
通常我們想要在odoo中使用SQL語法時，都在model當中使用以定義方法的方式:
```CMD
    def raw_sql(self):
        query = """
            SELECT
                id, name, author
	        FROM
                my_library;
        """
        self.env.cr.execute(query)
        print('self.env.cr.fetchall:', self.env.cr.fetchall())
```
我們有三種取值的方式`self.env.cr.fetchall()` `self.env.cr.fetchone()` `self.env.cr.dictfetchall()`
### ○ SQL view
當報表的格式比較特殊的時候，我們就會使用他:
```CMD
@api.model_cr
    def init(self):
        tools.drop_view_if_exists(self.env.cr, self._table)
        query = """
        (
            SELECT
                min(library.id) as id,
                create_uid,
                avg(input_number) AS average_input_number
            FROM
                my_library AS library
            GROUP BY library.create_uid
        );
        """
        self.env.cr.execute(query)
```
※使用原生SQL時要特別注意，因為它是直接對DB做操作，並沒有ORM這層。
# Many2one
多對一欄位，舉個例子，員工會有很多個，但是對應的公司老闆通常只有一個。
```CMD
class Employee(models.Model):
    _name = 'employee'
    _description = "employee's model"
    
    name = fields.Char(string='Name', required=True)
    employee_id = fields.Many2one('boss', string='Boss', required=True)
```
# Many2many
多對多欄位，舉例員工有很多個，員工所擁有的證照也不只一個。
```CMD
class License(models.Model):
    _name = 'license'
    _description = "Employee's license"
    
    name = fields.Char(string='Name', required=True)
    employee_license = fields.Many2many('employee', 'employee_license_tag', 'license_id', 'employee_id', string='License')
```
`employee_license_tag`為M2M會生成的table名稱，當中的欄位分別是`license_id``employee_id`
### ○parameter參數
* ondelete - 預設為set null(關聯刪除時，本欄設定為空)，定義關聯欄位刪除時
，本欄為如何動作。另外還有restricted(關聯刪除時，會跳出警告並阻止刪除)、cascade(關聯刪除，本欄一併刪除)
* context - 可以使用在導引或是預設值。
* domain - 可以透過她來條件篩選。
* auto_join - 可以透過它來下query，但由於它會繞過ORM直接對DB下指令，使用上要特別小心。
# One2many
在odoo中，其實並沒有O2M的專屬欄位，它只是反向M2O，所以想使用O2M必須要有對應的M2O。
```CMD
class Boss(models.Model):
    _name = 'boss'
    _description = "Boss's model"
    
    name = fields.Char(string='Name', required=True)
    boss_id = fields.One2many('employee', 'employee_id', string='Boss ID')
```
# Class_Inheritance
### 修改既有或是擴充model，record存在在同一個DB_Table當中:
當我們想對`hr.employee`的model增加一個欄位時:
#### `model.py`
```cmd
class HREmployee(models.Model):
    _name = 'hr.employee'
    _inherit = 'hr.employee'

    test_field = fields.Char(string='Test Field', required=True)
```
特別要注意的地方是，_name不能去做更改，同時_name也是可以不用宣告的，在class_inheritance中是不能更改的
，因為更改了就會變成Prototype_Inheritance，另外在`view.xml`中我們必須使用`<field inherit_id>`來新增`test_field`
# Prototype_Inheritance
### 繼承父類別並且不會影響父類別，擁有自己的DB_Table儲存record:
當我們想要一個已存在model，並且不想影響他的record時:
```CMD
class PrototypeMailThread(models.Model):
    _name = 'prototype.mail.thread'
    _description = 'Prototype Inheritance MailThread'
    _inherit = ['mail.thread']

    test_field = fields.Char(string='Test Field', required=True)
```
prototype_inheritance中，會擁有自己的DB_Table，除了繼承`mail.thread`外，其實基本上更創建一個新的addons的操作是一樣的
# delegation_inheritance
### 會擁有自己的DB_Table儲存record，同時影響父類別的DB_Table
```CMD
class News(models.Model):
    _name = 'news'
    _description = 'Delegation Inheritance Demo'
    _inherits = {'res.partner': 'partner_id'}

    partner_id = fields.Many2one('res.partner', string='Partner2', required=True, ondelete='cascade')
    first_name = fields.Char(string='First Name', size=25)
```
使用delegation_inheritance的時候特別要注意的地方就是在inherit`s`還有必須給他M2O的欄位。

# report
### ○ templates格式
# Qweb

    