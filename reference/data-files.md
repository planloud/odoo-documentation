# 数据文件

- 本文来自[Odoo 13官方文档之开发者文档](../README.md)系列文章

Odoo是重度数据驱动的，并且模块定义的很大部分是其所管理的各种记录的定义：UI (菜单和视图)、安全权限(访问权限和访问规则)、报表和普通数据均由记录所定义。

## 结构

在Odoo 中定义数据的主要方式是通过XML数据文件：主要的XML数据文件的结构如下：

- `odoo`根元素中的任意数量的操作元素

```
<!-- the root elements of the data file -->
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
  <operation/>
  ...
</odoo>
```

数据文件单独执行，操作仅能引用之前定义过的操作结果

如果数据文件的内容预计仅应用一次，可以指定 odoo 标记`noupdate` 将其设置为 1。如果文件中的一部分数据预计仅应用一次，可以把文件的这部分内容放到 <data noupdate=”1”> 作用域中。

```
<odoo>
    <data noupdate="1">
        <!-- Only loaded when installing the module (odoo-bin -i module) -->
        <operation/>
    </data>

    <!-- (Re)Loaded at install and update (odoo-bin -i/-u) -->
    <operation/>
</odoo>
```

## 核心操作



### `record`

`record`相应地定义或更新数据库记录，有以下属性：

- `model` (必传)

  待创建(或更新)的模型名称

- `id`

  这条记录的[外部标识符](https://www.odoo.com/documentation/13.0/glossary.html#term-external-identifier)。强烈推荐进行提供对于记录的创建，允许随后的修改或引用该记录的定义对于记录修改，为要修改的记录

- `context`

  在创建记录时所使用的上下文

- `forcecreate`

  在更新模型下是否在记录不存在时进行创建要求有一个 [外部id](https://www.odoo.com/documentation/13.0/glossary.html#term-external-id)，默认值为 `True`。

### `field`

每条记录可由`field` 标签组成， 定义在创建记录时所设置的值。 不带`field`的`record` 会使用所有默认值（创建时）或什么都不做（更新时）。

`field` 有一个必选属性`name` ，为所要设置的字段的名称，以及定义值本身的各种方法：

- 不提供

  如果不对字段提供任何值，会对字段设置一个隐式的 `False` 。可用于清除字段或避免对字段使用默认值。

- `search`

  对于[关联字段](https://alanhou.org/odoo-13-orm-api/#reference-fields-relational)，应当为一个字段模型中的 [域](https://alanhou.org/odoo-13-orm-api/#reference-orm-domains)。会运行该域，使用它搜索字段模型并设置搜索的结果为字段值。若字段为[`Many2one`](https://alanhou.org/odoo-13-orm-api/#odoo.fields.Many2one)类型我们仅使用第一个结果。

- `ref`

  若提供了`ref` 属性，其值必须为一个有效的[外部id](https://www.odoo.com/documentation/13.0/glossary.html#term-external-id)，会进行查找并设置为字段值。大多针对 [`Many2one`](https://alanhou.org/odoo-13-orm-api/#odoo.fields.Many2one) 及 [`Reference`](https://alanhou.org/odoo-13-orm-api/#odoo.fields.Reference) 字段

- `type`

  若提供了 `type` 属性，用于解析和转化字段的内容。字段的内容可通过使用`file`属性的外部文件或通过节点体提供内容。可用的类型有：`xml`, `html`提取`字段`的子内容为单个文档，运行通过表单`%(external_id)s`所指定的任意 [外部id](https://www.odoo.com/documentation/13.0/glossary.html#term-external-id)。`%%` 可用于输出实际的*%* 符号。`file`确保字段内容为当前模型中的有限文件路径，保存`*模块*,路径` 对作为字段值`char`不进行修改直接设置字段内容为字段值`base64`[base64](https://tools.ietf.org/html/rfc3548.html#section-3)-编码字段的内容，通过`file` 属性合并来加载如图片文件到附件中`int`转化字段的内容为整型并设置其作为字段值`float`转化字段的内容为浮点型并设置其作为字段值`list`, `tuple`应包含带有相同属性作为`field` 的任意数量 `value` 元素，每个元素解析为所生成元组或列表的一项，所生成的集合会设置为字段值

- `eval`

  对于前面方法不适用的情况，`eval` 属性只需运行所提供的Python表达式，并设置结果为字段值。运行上下文包含各种模型(`time`, `datetime`, `timedelta`, `relativedelta`)，一个解析[外部标识符](https://www.odoo.com/documentation/13.0/glossary.html#term-external-identifiers) (`ref`) 的函数及可用时针对当前字段的模型对象 (`obj`)

### `delete`

`delete`标签可删除任意数量前面定义的记录。有如下属性：

- `model` (必传)

  所指定应删除记录的模型

- `id`

  要删除记录的 [外部id](https://www.odoo.com/documentation/13.0/glossary.html#term-external-id)

- `search`

  查找要删除模型记录的[域](https://alanhou.org/odoo-13-orm-api/#reference-orm-domains)

`id` 及 `search` 是其独有的

### `function`

`function`标签通过所提供参数调用模型中的方法。它拥有两个必须参数 `model` 和 `name` ，分别指定模型和要调用的方法名。

可以使用`eval`提供参数 (应运行为一个调用方法所使用的参数序列) 或 `value` 元素(参见 `list` 值)。

```
<odoo>
    <data noupdate="1">
        <record name="partner_1" model="res.partner">
            <field name="name">Odude</field>
        </record>

        <function model="res.partner" name="send_inscription_notice"
            eval="[[ref('partner_1'), ref('partner_2')]]"/>

        <function model="res.users" name="send_vip_inscription_notice">
            <function eval="[[('vip','=',True)]]" model="res.partner" name="search"/>
        </function>
    </data>

    <record id="model_form_view" model="ir.ui.view">

    </record>
</odoo>
```



## 快捷标签

因为Odoo的一些重要结构模型很复杂又耗时，数据文件提供了更简短的替代方案，使用[记录标签](https://www.odoo.com/documentation/13.0/reference/data.html#reference-data-record)来进行定义：

### `menuitem`

通过一些默认值或fallback来定义 `ir.ui.menu` 记录：

- `parent`

  如果设置了`parent` 属性，应当为其它菜单项的 [外部id](https://www.odoo.com/documentation/13.0/glossary.html#term-external-id)，用作新项的父级若未提供 `parent` ，尝试解析 `name`属性为 `/`- 菜单名的分隔序列并在菜单等级中查找位置。在这一解析中，会自动创建中间菜单否则会将菜单定义为“顶级”菜单项(*非* 无父级的菜单)

- `name`

  若未指定`name` 属性，尝试从存在的关联动作获取菜单名。否则使用该记录的 `id`

- `groups`

  `groups` 属性解析为`res.groups`模型的逗号分隔的[外部标识符](https://www.odoo.com/documentation/13.0/glossary.html#term-external-identifiers)。若[外部标识符](https://www.odoo.com/documentation/13.0/glossary.html#term-external-identifier)有前缀减号(`-`)，会从菜单组中*删除*组

- `action`

  若指定，`action`属性应当为在打开菜单时所执行动作的[外部标识符](https://www.odoo.com/documentation/13.0/glossary.html#term-external-id)

- `id`

  菜单项的[外部id](https://www.odoo.com/documentation/13.0/glossary.html#term-external-id)



### `template`

创建一个[QWeb视图](https://www.odoo.com/documentation/13.0/reference/views.html#reference-views-qweb) ，仅要求有视图中的 `arch` 版块，并允许一些*可选*属性：

- `id`

  视图的[外部标识符](https://www.odoo.com/documentation/13.0/glossary.html#term-external-identifier)

- `name`, `inherit_id`, `priority`

  与 `ir.ui.view` 中的相应字段相同(nb: `inherit_id` 应为一个[外部标识符](https://www.odoo.com/documentation/13.0/glossary.html#term-external-identifier))

- `primary`

  若设置为`True` 并与 `inherit_id`合并， 定义该视图为主视图

- `groups`

  [外部标识符](https://www.odoo.com/documentation/13.0/glossary.html#term-external-identifiers)组的逗号分隔列表

- `page`

  若设置为 `"True"`，模型为一个网站页面 (可链接、可删除)

- `optional`

  `enabled` 或 `disabled`， 视图是否可（在网站界面中）禁用及其默认状态。如未进行设置，视图保持启用状态。

### `report`

使用一些模型值创建一条 [IrActionsReport](https://alanhou.org/odoo-13-actions/#reference-actions-report) 记录。

大多仅为 `ir.actions.report`中相应字段的代理属性，但也会自动在报表的`model`中的More菜单中创建菜单项。

## CSV数据文件

XML数据是灵活且自描述的，但在批量创建一些模型的简单记录时会非常啰嗦。

对于这种情况，数据文件也可以使用 [csv](https://en.wikipedia.org/wiki/Comma-separated_values)，常用于[访问权限](https://www.odoo.com/documentation/13.0/reference/security.html#reference-security-acl)：

- 文件名为 `*model_name*.csv`
- 第一行通过特殊字段`id`作为 [外部标识符](https://www.odoo.com/documentation/13.0/glossary.html#term-external-identifiers)（用于创建或更新）列出要写入的字段
- 然后每行新建一条记录

这里数据文件的前几行定义美国的州 `res.country.state.csv`

```
"id","country_id:id","name","code"
state_au_1,au,"Australian Capital Territory","ACT"
state_au_2,au,"New South Wales","NSW"
state_au_3,au,"Northern Territory","NT"
state_au_4,au,"Queensland","QLD"
state_au_5,au,"South Australia","SA"
state_au_6,au,"Tasmania","TAS"
state_au_7,au,"Victoria","VIC"
state_au_8,au,"Western Australia","WA"
state_us_1,us,"Alabama","AL"
state_us_2,us,"Alaska","AK"
state_us_3,us,"Arizona","AZ"
state_us_4,us,"Arkansas","AR"
state_us_5,us,"California","CA"
state_us_6,us,"Colorado","CO"
```

解析为更为可读的格式：

| id             | country_id:id | name                                                     | code  |
| -------------- | ------------- | -------------------------------------------------------- | ----- |
| state_au_1     | au            | Australian Capital Territory                             | ACT   |
| state_au_2     | au            | New South Wales                                          | NSW   |
| state_au_3     | au            | Northern Territory                                       | NT    |
| state_au_4     | au            | Queensland                                               | QLD   |
| state_au_5     | au            | South Australia                                          | SA    |
| state_au_6     | au            | Tasmania                                                 | TAS   |
| state_au_7     | au            | Victoria                                                 | VIC   |
| state_au_8     | au            | Western Australia                                        | WA    |
| state_us_1     | us            | Alabama                                                  | AL    |
| state_us_2     | us            | Alaska                                                   | AK    |
| state_us_3     | us            | Arizona                                                  | AZ    |
| state_us_4     | us            | Arkansas                                                 | AR    |
| state_us_5     | us            | California                                               | CA    |
| state_us_6     | us            | Colorado                                                 | CO    |
| state_us_7     | us            | Connecticut                                              | CT    |
| state_us_8     | us            | Delaware                                                 | DE    |
| state_us_9     | us            | District of Columbia                                     | DC    |
| state_us_10    | us            | Florida                                                  | FL    |
| state_us_11    | us            | Georgia                                                  | GA    |
| state_us_12    | us            | Hawaii                                                   | HI    |
| state_us_13    | us            | Idaho                                                    | ID    |
| state_us_14    | us            | Illinois                                                 | IL    |
| state_us_15    | us            | Indiana                                                  | IN    |
| state_us_16    | us            | Iowa                                                     | IA    |
| state_us_17    | us            | Kansas                                                   | KS    |
| state_us_18    | us            | Kentucky                                                 | KY    |
| state_us_19    | us            | Louisiana                                                | LA    |
| state_us_20    | us            | Maine                                                    | ME    |
| state_us_21    | us            | Montana                                                  | MT    |
| state_us_22    | us            | Nebraska                                                 | NE    |
| state_us_23    | us            | Nevada                                                   | NV    |
| state_us_24    | us            | New Hampshire                                            | NH    |
| state_us_25    | us            | New Jersey                                               | NJ    |
| state_us_26    | us            | New Mexico                                               | NM    |
| state_us_27    | us            | New York                                                 | NY    |
| state_us_28    | us            | North Carolina                                           | NC    |
| state_us_29    | us            | North Dakota                                             | ND    |
| state_us_30    | us            | Ohio                                                     | OH    |
| state_us_31    | us            | Oklahoma                                                 | OK    |
| state_us_32    | us            | Oregon                                                   | OR    |
| state_us_33    | us            | Maryland                                                 | MD    |
| state_us_34    | us            | Massachusetts                                            | MA    |
| state_us_35    | us            | Michigan                                                 | MI    |
| state_us_36    | us            | Minnesota                                                | MN    |
| state_us_37    | us            | Mississippi                                              | MS    |
| state_us_38    | us            | Missouri                                                 | MO    |
| state_us_39    | us            | Pennsylvania                                             | PA    |
| state_us_40    | us            | Rhode Island                                             | RI    |
| state_us_41    | us            | South Carolina                                           | SC    |
| state_us_42    | us            | South Dakota                                             | SD    |
| state_us_43    | us            | Tennessee                                                | TN    |
| state_us_44    | us            | Texas                                                    | TX    |
| state_us_45    | us            | Utah                                                     | UT    |
| state_us_46    | us            | Vermont                                                  | VT    |
| state_us_47    | us            | Virginia                                                 | VA    |
| state_us_48    | us            | Washington                                               | WA    |
| state_us_49    | us            | West Virginia                                            | WV    |
| state_us_50    | us            | Wisconsin                                                | WI    |
| state_us_51    | us            | Wyoming                                                  | WY    |
| state_us_as    | us            | American Samoa                                           | AS    |
| state_us_fm    | us            | Federated States of Micronesia                           | FM    |
| state_us_gu    | us            | Guam                                                     | GU    |
| state_us_mh    | us            | Marshall Islands                                         | MH    |
| state_us_mp    | us            | Northern Mariana Islands                                 | MP    |
| state_us_pw    | us            | Palau                                                    | PW    |
| state_us_pr    | us            | Puerto Rico                                              | PR    |
| state_us_vi    | us            | Virgin Islands                                           | VI    |
| state_us_aa    | us            | Armed Forces Americas                                    | AA    |
| state_us_ae    | us            | Armed Forces Europe                                      | AE    |
| state_us_ap    | us            | Armed Forces Pacific                                     | AP    |
| state_br_ac    | br            | Acre                                                     | AC    |
| state_br_al    | br            | Alagoas                                                  | AL    |
| state_br_ap    | br            | Amapá                                                    | AP    |
| state_br_am    | br            | Amazonas                                                 | AM    |
| state_br_ba    | br            | Bahia                                                    | BA    |
| state_br_ce    | br            | Ceará                                                    | CE    |
| state_br_df    | br            | Distrito Federal                                         | DF    |
| state_br_es    | br            | Espírito Santo                                           | ES    |
| state_br_go    | br            | Goiás                                                    | GO    |
| state_br_ma    | br            | Maranhão                                                 | MA    |
| state_br_mt    | br            | Mato Grosso                                              | MT    |
| state_br_ms    | br            | Mato Grosso do Sul                                       | MS    |
| state_br_mg    | br            | Minas Gerais                                             | MG    |
| state_br_pa    | br            | Pará                                                     | PA    |
| state_br_pb    | br            | Paraíba                                                  | PB    |
| state_br_pr    | br            | Paraná                                                   | PR    |
| state_br_pe    | br            | Pernambuco                                               | PE    |
| state_br_pi    | br            | Piauí                                                    | PI    |
| state_br_rj    | br            | Rio de Janeiro                                           | RJ    |
| state_br_rn    | br            | Rio Grande do Norte                                      | RN    |
| state_br_rs    | br            | Rio Grande do Sul                                        | RS    |
| state_br_ro    | br            | Rondônia                                                 | RO    |
| state_br_rr    | br            | Roraima                                                  | RR    |
| state_br_sc    | br            | Santa Catarina                                           | SC    |
| state_br_sp    | br            | São Paulo                                                | SP    |
| state_br_se    | br            | Sergipe                                                  | SE    |
| state_br_to    | br            | Tocantins                                                | TO    |
| state_ru_ad    | ru            | Republic of Adygeya                                      | AD    |
| state_ru_al    | ru            | Altai Republic                                           | AL    |
| state_ru_alt   | ru            | Altai Krai                                               | ALT   |
| state_ru_amu   | ru            | Amur Oblast                                              | AMU   |
| state_ru_ark   | ru            | Arkhangelsk Oblast                                       | ARK   |
| state_ru_ast   | ru            | Astrakhan Oblast                                         | AST   |
| state_ru_ba    | ru            | Republic of Bashkortostan                                | BA    |
| state_ru_bel   | ru            | Belgorod Oblast                                          | BEL   |
| state_ru_bry   | ru            | Bryansk Oblast                                           | BRY   |
| state_ru_bu    | ru            | Republic of Buryatia                                     | BU    |
| state_ru_ce    | ru            | Chechen Republic                                         | CE    |
| state_ru_che   | ru            | Chelyabinsk Oblast                                       | CHE   |
| state_ru_chu   | ru            | Chukotka Autonomous Okrug                                | CHU   |
| state_ru_cu    | ru            | Chuvash Republic                                         | CU    |
| state_ru_da    | ru            | Republic of Dagestan                                     | DA    |
| state_ru_in    | ru            | Republic of Ingushetia                                   | IN    |
| state_ru_irk   | ru            | Irkutsk Oblast                                           | IRK   |
| state_ru_iva   | ru            | Ivanovo Oblast                                           | IVA   |
| state_ru_kam   | ru            | Kamchatka Krai                                           | KAM   |
| state_ru_kb    | ru            | Kabardino-Balkarian Republic                             | KB    |
| state_ru_kgd   | ru            | Kaliningrad Oblast                                       | KGD   |
| state_ru_kl    | ru            | Republic of Kalmykia                                     | KL    |
| state_ru_klu   | ru            | Kaluga Oblast                                            | KLU   |
| state_ru_kc    | ru            | Karachay–Cherkess Republic                               | KC    |
| state_ru_kr    | ru            | Republic of Karelia                                      | KR    |
| state_ru_kem   | ru            | Kemerovo Oblast                                          | KEM   |
| state_ru_kha   | ru            | Khabarovsk Krai                                          | KHA   |
| state_ru_kk    | ru            | Republic of Khakassia                                    | KK    |
| state_ru_khm   | ru            | Khanty-Mansi Autonomous Okrug                            | KHM   |
| state_ru_kir   | ru            | Kirov Oblast                                             | KIR   |
| state_ru_ko    | ru            | Komi Republic                                            | KO    |
| state_ru_kos   | ru            | Kostroma Oblast                                          | KOS   |
| state_ru_kda   | ru            | Krasnodar Krai                                           | KDA   |
| state_ru_kya   | ru            | Krasnoyarsk Krai                                         | KYA   |
| state_ru_kgn   | ru            | Kurgan Oblast                                            | KGN   |
| state_ru_krs   | ru            | Kursk Oblast                                             | KRS   |
| state_ru_len   | ru            | Leningrad Oblast                                         | LEN   |
| state_ru_lip   | ru            | Lipetsk Oblast                                           | LIP   |
| state_ru_mag   | ru            | Magadan Oblast                                           | MAG   |
| state_ru_me    | ru            | Mari El Republic                                         | ME    |
| state_ru_mo    | ru            | Republic of Mordovia                                     | MO    |
| state_ru_mos   | ru            | Moscow Oblast                                            | MOS   |
| state_ru_mow   | ru            | Moscow                                                   | MOW   |
| state_ru_mur   | ru            | Murmansk Oblast                                          | MUR   |
| state_ru_niz   | ru            | Nizhny Novgorod Oblast                                   | NIZ   |
| state_ru_ngr   | ru            | Novgorod Oblast                                          | NGR   |
| state_ru_nvs   | ru            | Novosibirsk Oblast                                       | NVS   |
| state_ru_oms   | ru            | Omsk Oblast                                              | OMS   |
| state_ru_ore   | ru            | Orenburg Oblast                                          | ORE   |
| state_ru_orl   | ru            | Oryol Oblast                                             | ORL   |
| state_ru_pnz   | ru            | Penza Oblast                                             | PNZ   |
| state_ru_per   | ru            | Perm Krai                                                | PER   |
| state_ru_pri   | ru            | Primorsky Krai                                           | PRI   |
| state_ru_psk   | ru            | Pskov Oblast                                             | PSK   |
| state_ru_ros   | ru            | Rostov Oblast                                            | ROS   |
| state_ru_rya   | ru            | Ryazan Oblast                                            | RYA   |
| state_ru_sa    | ru            | Sakha Republic (Yakutia)                                 | SA    |
| state_ru_sak   | ru            | Sakhalin Oblast                                          | SAK   |
| state_ru_sam   | ru            | Samara Oblast                                            | SAM   |
| state_ru_spe   | ru            | Saint Petersburg                                         | SPE   |
| state_ru_sar   | ru            | Saratov Oblast                                           | SAR   |
| state_ru_se    | ru            | Republic of North Ossetia–Alania                         | SE    |
| state_ru_smo   | ru            | Smolensk Oblast                                          | SMO   |
| state_ru_sta   | ru            | Stavropol Krai                                           | STA   |
| state_ru_sve   | ru            | Sverdlovsk Oblast                                        | SVE   |
| state_ru_tam   | ru            | Tambov Oblast                                            | TAM   |
| state_ru_ta    | ru            | Republic of Tatarstan                                    | TA    |
| state_ru_tom   | ru            | Tomsk Oblast                                             | TOM   |
| state_ru_tul   | ru            | Tula Oblast                                              | TUL   |
| state_ru_tve   | ru            | Tver Oblast                                              | TVE   |
| state_ru_tyu   | ru            | Tyumen Oblast                                            | TYU   |
| state_ru_ty    | ru            | Tyva Republic                                            | TY    |
| state_ru_ud    | ru            | Udmurtia                                                 | UD    |
| state_ru_uly   | ru            | Ulyanovsk Oblast                                         | ULY   |
| state_ru_vla   | ru            | Vladimir Oblast                                          | VLA   |
| state_ru_vgg   | ru            | Volgograd Oblast                                         | VGG   |
| state_ru_vlg   | ru            | Vologda Oblast                                           | VLG   |
| state_ru_vor   | ru            | Voronezh Oblast                                          | VOR   |
| state_ru_yan   | ru            | Yamalo-Nenets Autonomous Okrug                           | YAN   |
| state_ru_yar   | ru            | Yaroslavl Oblast                                         | YAR   |
| state_ru_yev   | ru            | Jewish Autonomous Oblast                                 | YEV   |
| state_gt_ave   | gt            | Alta Verapaz                                             | AVE   |
| state_gt_bve   | gt            | Baja Verapaz                                             | BVE   |
| state_gt_cmt   | gt            | Chimaltenango                                            | CMT   |
| state_gt_cqm   | gt            | Chiquimula                                               | CQM   |
| state_gt_epr   | gt            | El Progreso                                              | EPR   |
| state_gt_esc   | gt            | Escuintla                                                | ESC   |
| state_gt_gua   | gt            | Guatemala                                                | GUA   |
| state_gt_hue   | gt            | Huehuetenango                                            | HUE   |
| state_gt_iza   | gt            | Izabal                                                   | IZA   |
| state_gt_jal   | gt            | Jalapa                                                   | JAL   |
| state_gt_jut   | gt            | Jutiapa                                                  | JUT   |
| state_gt_pet   | gt            | Petén                                                    | PET   |
| state_gt_que   | gt            | Quetzaltenango                                           | QUE   |
| state_gt_qui   | gt            | Quiché                                                   | QUI   |
| state_gt_ret   | gt            | Retalhuleu                                               | RET   |
| state_gt_sac   | gt            | Sacatepéquez                                             | SAC   |
| state_gt_sma   | gt            | San Marcos                                               | SMA   |
| state_gt_sro   | gt            | Santa Rosa                                               | SRO   |
| state_gt_sol   | gt            | Sololá                                                   | SOL   |
| state_gt_suc   | gt            | Suchitepéquez                                            | SUC   |
| state_gt_tot   | gt            | Totonicapán                                              | TOT   |
| state_gt_zac   | gt            | Zacapa                                                   | ZAC   |
| state_jp_jp-23 | jp            | Aichi                                                    | 23    |
| state_jp_jp-05 | jp            | Akita                                                    | 05    |
| state_jp_jp-02 | jp            | Aomori                                                   | 02    |
| state_jp_jp-12 | jp            | Chiba                                                    | 12    |
| state_jp_jp-38 | jp            | Ehime                                                    | 38    |
| state_jp_jp-18 | jp            | Fukui                                                    | 18    |
| state_jp_jp-40 | jp            | Fukuoka                                                  | 40    |
| state_jp_jp-07 | jp            | Fukushima                                                | 07    |
| state_jp_jp-21 | jp            | Gifu                                                     | 21    |
| state_jp_jp-10 | jp            | Gunma                                                    | 10    |
| state_jp_jp-34 | jp            | Hiroshima                                                | 34    |
| state_jp_jp-01 | jp            | Hokkaidō                                                 | 01    |
| state_jp_jp-28 | jp            | Hyōgo                                                    | 28    |
| state_jp_jp-08 | jp            | Ibaraki                                                  | 08    |
| state_jp_jp-17 | jp            | Ishikawa                                                 | 17    |
| state_jp_jp-03 | jp            | Iwate                                                    | 03    |
| state_jp_jp-37 | jp            | Kagawa                                                   | 37    |
| state_jp_jp-46 | jp            | Kagoshima                                                | 46    |
| state_jp_jp-14 | jp            | Kanagawa                                                 | 14    |
| state_jp_jp-39 | jp            | Kōchi                                                    | 39    |
| state_jp_jp-43 | jp            | Kumamoto                                                 | 43    |
| state_jp_jp-26 | jp            | Kyōto                                                    | 26    |
| state_jp_jp-24 | jp            | Mie                                                      | 24    |
| state_jp_jp-04 | jp            | Miyagi                                                   | 04    |
| state_jp_jp-45 | jp            | Miyazaki                                                 | 45    |
| state_jp_jp-20 | jp            | Nagano                                                   | 20    |
| state_jp_jp-42 | jp            | Nagasaki                                                 | 42    |
| state_jp_jp-29 | jp            | Nara                                                     | 29    |
| state_jp_jp-15 | jp            | Niigata                                                  | 15    |
| state_jp_jp-44 | jp            | Ōita                                                     | 44    |
| state_jp_jp-33 | jp            | Okayama                                                  | 33    |
| state_jp_jp-47 | jp            | Okinawa                                                  | 47    |
| state_jp_jp-27 | jp            | Ōsaka                                                    | 27    |
| state_jp_jp-41 | jp            | Saga                                                     | 41    |
| state_jp_jp-11 | jp            | Saitama                                                  | 11    |
| state_jp_jp-25 | jp            | Shiga                                                    | 25    |
| state_jp_jp-32 | jp            | Shimane                                                  | 32    |
| state_jp_jp-22 | jp            | Shizuoka                                                 | 22    |
| state_jp_jp-09 | jp            | Tochigi                                                  | 09    |
| state_jp_jp-36 | jp            | Tokushima                                                | 36    |
| state_jp_jp-31 | jp            | Tottori                                                  | 31    |
| state_jp_jp-16 | jp            | Toyama                                                   | 16    |
| state_jp_jp-13 | jp            | Tōkyō                                                    | 13    |
| state_jp_jp-30 | jp            | Wakayama                                                 | 30    |
| state_jp_jp-06 | jp            | Yamagata                                                 | 06    |
| state_jp_jp-35 | jp            | Yamaguchi                                                | 35    |
| state_jp_jp-19 | jp            | Yamanashi                                                | 19    |
| state_pt_pt-01 | pt            | Aveiro                                                   | 01    |
| state_pt_pt-02 | pt            | Beja                                                     | 02    |
| state_pt_pt-03 | pt            | Braga                                                    | 03    |
| state_pt_pt-04 | pt            | Bragança                                                 | 04    |
| state_pt_pt-05 | pt            | Castelo Branco                                           | 05    |
| state_pt_pt-06 | pt            | Coimbra                                                  | 06    |
| state_pt_pt-07 | pt            | Évora                                                    | 07    |
| state_pt_pt-08 | pt            | Faro                                                     | 08    |
| state_pt_pt-09 | pt            | Guarda                                                   | 09    |
| state_pt_pt-10 | pt            | Leiria                                                   | 10    |
| state_pt_pt-11 | pt            | Lisboa                                                   | 11    |
| state_pt_pt-12 | pt            | Portalegre                                               | 12    |
| state_pt_pt-13 | pt            | Porto                                                    | 13    |
| state_pt_pt-14 | pt            | Santarém                                                 | 14    |
| state_pt_pt-15 | pt            | Setúbal                                                  | 15    |
| state_pt_pt-16 | pt            | Viana do Castelo                                         | 16    |
| state_pt_pt-17 | pt            | Vila Real                                                | 17    |
| state_pt_pt-18 | pt            | Viseu                                                    | 18    |
| state_pt_pt-20 | pt            | Açores                                                   | 20    |
| state_pt_pt-30 | pt            | Madeira                                                  | 30    |
| state_eg_dk    | eg            | Dakahlia                                                 | DK    |
| state_eg_ba    | eg            | Red Sea                                                  | BA    |
| state_eg_bh    | eg            | Beheira                                                  | BH    |
| state_eg_fym   | eg            | Faiyum                                                   | FYM   |
| state_eg_gh    | eg            | Gharbia                                                  | GH    |
| state_eg_alx   | eg            | Alexandria                                               | ALX   |
| state_eg_is    | eg            | Ismailia                                                 | IS    |
| state_eg_gz    | eg            | Giza                                                     | GZ    |
| state_eg_mnf   | eg            | Monufia                                                  | MNF   |
| state_eg_mn    | eg            | Minya                                                    | MN    |
| state_eg_c     | eg            | Cairo                                                    | C     |
| state_eg_kb    | eg            | Qalyubia                                                 | KB    |
| state_eg_lx    | eg            | Luxor                                                    | LX    |
| state_eg_wad   | eg            | New Valley                                               | WAD   |
| state_eg_shr   | eg            | Al Sharqia                                               | SHR   |
| state_eg_su    | eg            | 6th of October                                           | SU    |
| state_eg_suz   | eg            | Suez                                                     | SUZ   |
| state_eg_asn   | eg            | Aswan                                                    | ASN   |
| state_eg_ast   | eg            | Asyut                                                    | AST   |
| state_eg_bns   | eg            | Beni Suef                                                | BNS   |
| state_eg_pts   | eg            | Port Said                                                | PTS   |
| state_eg_dt    | eg            | Damietta                                                 | DT    |
| state_eg_hu    | eg            | Helwan                                                   | HU    |
| state_eg_js    | eg            | South Sinai                                              | JS    |
| state_eg_kfs   | eg            | Kafr el-Sheikh                                           | KFS   |
| state_eg_mt    | eg            | Matrouh                                                  | MT    |
| state_eg_kn    | eg            | Qena                                                     | KN    |
| state_eg_sin   | eg            | North Sinai                                              | SIN   |
| state_eg_shg   | eg            | Sohag                                                    | SHG   |
| state_za_ec    | za            | Eastern Cape                                             | EC    |
| state_za_fs    | za            | Free State                                               | FS    |
| state_za_gt    | za            | Gauteng                                                  | GT    |
| state_za_nl    | za            | KwaZulu-Natal                                            | NL    |
| state_za_lp    | za            | Limpopo                                                  | LP    |
| state_za_mp    | za            | Mpumalanga                                               | MP    |
| state_za_nc    | za            | Northern Cape                                            | NC    |
| state_za_nw    | za            | North West                                               | NW    |
| state_za_wc    | za            | Western Cape                                             | WC    |
| state_it_ag    | it            | Agrigento                                                | AG    |
| state_it_al    | it            | Alessandria                                              | AL    |
| state_it_an    | it            | Ancona                                                   | AN    |
| state_it_ao    | it            | Aosta                                                    | AO    |
| state_it_ar    | it            | Arezzo                                                   | AR    |
| state_it_ap    | it            | Ascoli Piceno                                            | AP    |
| state_it_at    | it            | Asti                                                     | AT    |
| state_it_av    | it            | Avellino                                                 | AV    |
| state_it_ba    | it            | Bari                                                     | BA    |
| state_it_bt    | it            | Barletta-Andria-Trani                                    | BT    |
| state_it_bl    | it            | Belluno                                                  | BL    |
| state_it_bn    | it            | Benevento                                                | BN    |
| state_it_bg    | it            | Bergamo                                                  | BG    |
| state_it_bi    | it            | Biella                                                   | BI    |
| state_it_bo    | it            | Bologna                                                  | BO    |
| state_it_bz    | it            | Bolzano                                                  | BZ    |
| state_it_bs    | it            | Brescia                                                  | BS    |
| state_it_br    | it            | Brindisi                                                 | BR    |
| state_it_ca    | it            | Cagliari                                                 | CA    |
| state_it_cl    | it            | Caltanissetta                                            | CL    |
| state_it_cb    | it            | Campobasso                                               | CB    |
| state_it_ci    | it            | Carbonia-Iglesias                                        | CI    |
| state_it_ce    | it            | Caserta                                                  | CE    |
| state_it_ct    | it            | Catania                                                  | CT    |
| state_it_cz    | it            | Catanzaro                                                | CZ    |
| state_it_ch    | it            | Chieti                                                   | CH    |
| state_it_co    | it            | Como                                                     | CO    |
| state_it_cs    | it            | Cosenza                                                  | CS    |
| state_it_cr    | it            | Cremona                                                  | CR    |
| state_it_kr    | it            | Crotone                                                  | KR    |
| state_it_cn    | it            | Cuneo                                                    | CN    |
| state_it_en    | it            | Enna                                                     | EN    |
| state_it_fm    | it            | Fermo                                                    | FM    |
| state_it_fe    | it            | Ferrara                                                  | FE    |
| state_it_fi    | it            | Firenze                                                  | FI    |
| state_it_fg    | it            | Foggia                                                   | FG    |
| state_it_fc    | it            | Forlì-Cesena                                             | FC    |
| state_it_fr    | it            | Frosinone                                                | FR    |
| state_it_ge    | it            | Genova                                                   | GE    |
| state_it_go    | it            | Gorizia                                                  | GO    |
| state_it_gr    | it            | Grosseto                                                 | GR    |
| state_it_im    | it            | Imperia                                                  | IM    |
| state_it_is    | it            | Isernia                                                  | IS    |
| state_it_sp    | it            | La Spezia                                                | SP    |
| state_it_aq    | it            | L’Aquila                                                 | AQ    |
| state_it_lt    | it            | Latina                                                   | LT    |
| state_it_le    | it            | Lecce                                                    | LE    |
| state_it_lc    | it            | Lecco                                                    | LC    |
| state_it_li    | it            | Livorno                                                  | LI    |
| state_it_lo    | it            | Lodi                                                     | LO    |
| state_it_lu    | it            | Lucca                                                    | LU    |
| state_it_mc    | it            | Macerata                                                 | MC    |
| state_it_mn    | it            | Mantova                                                  | MN    |
| state_it_ms    | it            | Massa-Carrara                                            | MS    |
| state_it_mt    | it            | Matera                                                   | MT    |
| state_it_vs    | it            | Medio Campidano                                          | VS    |
| state_it_me    | it            | Messina                                                  | ME    |
| state_it_mi    | it            | Milano                                                   | MI    |
| state_it_mo    | it            | Modena                                                   | MO    |
| state_it_mb    | it            | Monza e Brianza                                          | MB    |
| state_it_na    | it            | Napoli                                                   | NA    |
| state_it_no    | it            | Novara                                                   | NO    |
| state_it_nu    | it            | Nuoro                                                    | NU    |
| state_it_og    | it            | Ogliastra                                                | OG    |
| state_it_ot    | it            | Olbia-Tempio                                             | OT    |
| state_it_or    | it            | Oristano                                                 | OR    |
| state_it_pd    | it            | Padova                                                   | PD    |
| state_it_pa    | it            | Palermo                                                  | PA    |
| state_it_pr    | it            | Parma                                                    | PR    |
| state_it_pv    | it            | Pavia                                                    | PV    |
| state_it_pg    | it            | Perugia                                                  | PG    |
| state_it_pu    | it            | Pesaro e Urbino                                          | PU    |
| state_it_pe    | it            | Pescara                                                  | PE    |
| state_it_pc    | it            | Piacenza                                                 | PC    |
| state_it_pi    | it            | Pisa                                                     | PI    |
| state_it_pt    | it            | Pistoia                                                  | PT    |
| state_it_pn    | it            | Pordenone                                                | PN    |
| state_it_pz    | it            | Potenza                                                  | PZ    |
| state_it_po    | it            | Prato                                                    | PO    |
| state_it_rg    | it            | Ragusa                                                   | RG    |
| state_it_ra    | it            | Ravenna                                                  | RA    |
| state_it_rc    | it            | Reggio Calabria                                          | RC    |
| state_it_re    | it            | Reggio Emilia                                            | RE    |
| state_it_ri    | it            | Rieti                                                    | RI    |
| state_it_rn    | it            | Rimini                                                   | RN    |
| state_it_rm    | it            | Roma                                                     | RM    |
| state_it_ro    | it            | Rovigo                                                   | RO    |
| state_it_sa    | it            | Salerno                                                  | SA    |
| state_it_ss    | it            | Sassari                                                  | SS    |
| state_it_sv    | it            | Savona                                                   | SV    |
| state_it_si    | it            | Siena                                                    | SI    |
| state_it_sr    | it            | Siracusa                                                 | SR    |
| state_it_so    | it            | Sondrio                                                  | SO    |
| state_it_su    | it            | Sud Sardegna                                             | SU    |
| state_it_ta    | it            | Taranto                                                  | TA    |
| state_it_te    | it            | Teramo                                                   | TE    |
| state_it_tr    | it            | Terni                                                    | TR    |
| state_it_to    | it            | Torino                                                   | TO    |
| state_it_tp    | it            | Trapani                                                  | TP    |
| state_it_tn    | it            | Trento                                                   | TN    |
| state_it_tv    | it            | Treviso                                                  | TV    |
| state_it_ts    | it            | Trieste                                                  | TS    |
| state_it_ud    | it            | Udine                                                    | UD    |
| state_it_va    | it            | Varese                                                   | VA    |
| state_it_ve    | it            | Venezia                                                  | VE    |
| state_it_vb    | it            | Verbano-Cusio-Ossola                                     | VB    |
| state_it_vc    | it            | Vercelli                                                 | VC    |
| state_it_vr    | it            | Verona                                                   | VR    |
| state_it_vv    | it            | Vibo Valentia                                            | VV    |
| state_it_vi    | it            | Vicenza                                                  | VI    |
| state_it_vt    | it            | Viterbo                                                  | VT    |
| state_es_c     | es            | A Coruña (La Coruña)                                     | C     |
| state_es_vi    | es            | Araba/Álava                                              | VI    |
| state_es_ab    | es            | Albacete                                                 | AB    |
| state_es_a     | es            | Alacant (Alicante)                                       | A     |
| state_es_al    | es            | Almería                                                  | AL    |
| state_es_o     | es            | Asturias                                                 | O     |
| state_es_av    | es            | Ávila                                                    | AV    |
| state_es_ba    | es            | Badajoz                                                  | BA    |
| state_es_pm    | es            | Illes Balears (Islas Baleares)                           | PM    |
| state_es_b     | es            | Barcelona                                                | B     |
| state_es_bu    | es            | Burgos                                                   | BU    |
| state_es_cc    | es            | Cáceres                                                  | CC    |
| state_es_ca    | es            | Cádiz                                                    | CA    |
| state_es_s     | es            | Cantabria                                                | S     |
| state_es_cs    | es            | Castelló (Castellón)                                     | CS    |
| state_es_ce    | es            | Ceuta                                                    | CE    |
| state_es_cr    | es            | Ciudad Real                                              | CR    |
| state_es_co    | es            | Córdoba                                                  | CO    |
| state_es_cu    | es            | Cuenca                                                   | CU    |
| state_es_gi    | es            | Girona (Gerona)                                          | GI    |
| state_es_gr    | es            | Granada                                                  | GR    |
| state_es_gu    | es            | Guadalajara                                              | GU    |
| state_es_ss    | es            | Gipuzkoa (Guipúzcoa)                                     | SS    |
| state_es_h     | es            | Huelva                                                   | H     |
| state_es_hu    | es            | Huesca                                                   | HU    |
| state_es_j     | es            | Jaén                                                     | J     |
| state_es_lo    | es            | La Rioja                                                 | LO    |
| state_es_gc    | es            | Las Palmas                                               | GC    |
| state_es_le    | es            | León                                                     | LE    |
| state_es_l     | es            | Lleida (Lérida)                                          | L     |
| state_es_lu    | es            | Lugo                                                     | LU    |
| state_es_m     | es            | Madrid                                                   | M     |
| state_es_ma    | es            | Málaga                                                   | MA    |
| state_es_ml    | es            | Melilla                                                  | ME    |
| state_es_mu    | es            | Murcia                                                   | MU    |
| state_es_na    | es            | Navarra (Nafarroa)                                       | NA    |
| state_es_or    | es            | Ourense (Orense)                                         | OR    |
| state_es_p     | es            | Palencia                                                 | P     |
| state_es_po    | es            | Pontevedra                                               | PO    |
| state_es_sa    | es            | Salamanca                                                | SA    |
| state_es_tf    | es            | Santa Cruz de Tenerife                                   | TF    |
| state_es_sg    | es            | Segovia                                                  | SG    |
| state_es_se    | es            | Sevilla                                                  | SE    |
| state_es_so    | es            | Soria                                                    | SO    |
| state_es_t     | es            | Tarragona                                                | T     |
| state_es_te    | es            | Teruel                                                   | TE    |
| state_es_to    | es            | Toledo                                                   | TO    |
| state_es_v     | es            | València (Valencia)                                      | V     |
| state_es_va    | es            | Valladolid                                               | VA    |
| state_es_bi    | es            | Bizkaia (Vizcaya)                                        | BI    |
| state_es_za    | es            | Zamora                                                   | ZA    |
| state_es_z     | es            | Zaragoza                                                 | Z     |
| state_my_jhr   | my            | Johor                                                    | JHR   |
| state_my_kdh   | my            | Kedah                                                    | KDH   |
| state_my_ktn   | my            | Kelantan                                                 | KTN   |
| state_my_kul   | my            | Kuala Lumpur                                             | KUL   |
| state_my_lbn   | my            | Labuan                                                   | LBN   |
| state_my_mlk   | my            | Melaka                                                   | MLK   |
| state_my_nsn   | my            | Negeri Sembilan                                          | NSN   |
| state_my_phg   | my            | Pahang                                                   | PHG   |
| state_my_prk   | my            | Perak                                                    | PRK   |
| state_my_pls   | my            | Perlis                                                   | PLS   |
| state_my_png   | my            | Pulau Pinang                                             | PNG   |
| state_my_pjy   | my            | Putrajaya                                                | PJY   |
| state_my_sbh   | my            | Sabah                                                    | SBH   |
| state_my_swk   | my            | Sarawak                                                  | SWK   |
| state_my_sgr   | my            | Selangor                                                 | SGR   |
| state_my_trg   | my            | Terengganu                                               | TRG   |
| state_mx_ags   | mx            | Aguascalientes                                           | AGU   |
| state_mx_bc    | mx            | Baja California                                          | BCN   |
| state_mx_bcs   | mx            | Baja California Sur                                      | BCS   |
| state_mx_chih  | mx            | Chihuahua                                                | CHH   |
| state_mx_col   | mx            | Colima                                                   | COL   |
| state_mx_camp  | mx            | Campeche                                                 | CAM   |
| state_mx_coah  | mx            | Coahuila                                                 | COA   |
| state_mx_chis  | mx            | Chiapas                                                  | CHP   |
| state_mx_df    | mx            | Ciudad de México                                         | DIF   |
| state_mx_dgo   | mx            | Durango                                                  | DUR   |
| state_mx_gro   | mx            | Guerrero                                                 | GRO   |
| state_mx_gto   | mx            | Guanajuato                                               | GUA   |
| state_mx_hgo   | mx            | Hidalgo                                                  | HID   |
| state_mx_jal   | mx            | Jalisco                                                  | JAL   |
| state_mx_mich  | mx            | Michoacán                                                | MIC   |
| state_mx_mor   | mx            | Morelos                                                  | MOR   |
| state_mx_mex   | mx            | México                                                   | MEX   |
| state_mx_nay   | mx            | Nayarit                                                  | NAY   |
| state_mx_nl    | mx            | Nuevo León                                               | NLE   |
| state_mx_oax   | mx            | Oaxaca                                                   | OAX   |
| state_mx_pue   | mx            | Puebla                                                   | PUE   |
| state_mx_q_roo | mx            | Quintana Roo                                             | ROO   |
| state_mx_qro   | mx            | Querétaro                                                | QUE   |
| state_mx_sin   | mx            | Sinaloa                                                  | SIN   |
| state_mx_slp   | mx            | San Luis Potosí                                          | SLP   |
| state_mx_son   | mx            | Sonora                                                   | SON   |
| state_mx_tab   | mx            | Tabasco                                                  | TAB   |
| state_mx_tlax  | mx            | Tlaxcala                                                 | TLA   |
| state_mx_tamps | mx            | Tamaulipas                                               | TAM   |
| state_mx_ver   | mx            | Veracruz                                                 | VER   |
| state_mx_yuc   | mx            | Yucatán                                                  | YUC   |
| state_mx_zac   | mx            | Zacatecas                                                | ZAC   |
| state_nz_auk   | nz            | Auckland                                                 | AUK   |
| state_nz_bop   | nz            | Bay of Plenty                                            | BOP   |
| state_nz_can   | nz            | Canterbury                                               | CAN   |
| state_nz_gis   | nz            | Gisborne                                                 | GIS   |
| state_nz_hkb   | nz            | Hawke’s Bay                                              | HKB   |
| state_nz_mwt   | nz            | Manawatu-Wanganui                                        | MWT   |
| state_nz_mbh   | nz            | Marlborough                                              | MBH   |
| state_nz_nsn   | nz            | Nelson                                                   | NSN   |
| state_nz_ntl   | nz            | Northland                                                | NTL   |
| state_nz_ota   | nz            | Otago                                                    | OTA   |
| state_nz_stl   | nz            | Southland                                                | STL   |
| state_nz_tki   | nz            | Taranaki                                                 | TKI   |
| state_nz_tas   | nz            | Tasman                                                   | TAS   |
| state_nz_wko   | nz            | Waikato                                                  | WKO   |
| state_nz_wgn   | nz            | Wellington                                               | WGN   |
| state_nz_wtc   | nz            | West Coast                                               | WTC   |
| state_ca_ab    | ca            | Alberta                                                  | AB    |
| state_ca_bc    | ca            | British Columbia                                         | BC    |
| state_ca_mb    | ca            | Manitoba                                                 | MB    |
| state_ca_nb    | ca            | New Brunswick                                            | NB    |
| state_ca_nl    | ca            | Newfoundland and Labrador                                | NL    |
| state_ca_nt    | ca            | Northwest Territories                                    | NT    |
| state_ca_ns    | ca            | Nova Scotia                                              | NS    |
| state_ca_nu    | ca            | Nunavut                                                  | NU    |
| state_ca_on    | ca            | Ontario                                                  | ON    |
| state_ca_pe    | ca            | Prince Edward Island                                     | PE    |
| state_ca_qc    | ca            | Quebec                                                   | QC    |
| state_ca_sk    | ca            | Saskatchewan                                             | SK    |
| state_ca_yt    | ca            | Yukon                                                    | YT    |
| state_ae_az    | ae            | Abu Dhabi                                                | AZ    |
| state_ae_aj    | ae            | Ajman                                                    | AJ    |
| state_ae_du    | ae            | Dubai                                                    | DU    |
| state_ae_fu    | ae            | Fujairah                                                 | FU    |
| state_ae_rk    | ae            | Ras al-Khaimah                                           | RK    |
| state_ae_sh    | ae            | Sharjah                                                  | SH    |
| state_ae_uq    | ae            | Umm al-Quwain                                            | UQ    |
| state_ar_c     | ar            | Ciudad Autónoma de Buenos Aires                          | C     |
| state_ar_b     | ar            | Buenos Aires                                             | B     |
| state_ar_k     | ar            | Catamarca                                                | K     |
| state_ar_h     | ar            | Chaco                                                    | H     |
| state_ar_u     | ar            | Chobut                                                   | U     |
| state_ar_x     | ar            | Córdoba                                                  | X     |
| state_ar_w     | ar            | Corrientes                                               | W     |
| state_ar_e     | ar            | Entre Ríos                                               | E     |
| state_ar_p     | ar            | Formosa                                                  | P     |
| state_ar_y     | ar            | Jujuy                                                    | Y     |
| state_ar_l     | ar            | La Pampa                                                 | L     |
| state_ar_f     | ar            | La Rioja                                                 | F     |
| state_ar_m     | ar            | Mendoza                                                  | M     |
| state_ar_n     | ar            | Misiones                                                 | N     |
| state_ar_q     | ar            | Neuquén                                                  | Q     |
| state_ar_r     | ar            | Río Negro                                                | R     |
| state_ar_a     | ar            | Salta                                                    | A     |
| state_ar_j     | ar            | San Juan                                                 | J     |
| state_ar_d     | ar            | San Luis                                                 | D     |
| state_ar_z     | ar            | Santa Cruz                                               | Z     |
| state_ar_s     | ar            | Santa Fe                                                 | S     |
| state_ar_g     | ar            | Santiago Del Estero                                      | G     |
| state_ar_v     | ar            | Tierra del Fuego                                         | V     |
| state_ar_t     | ar            | Tucumán                                                  | T     |
| state_in_an    | in            | Andaman and Nicobar                                      | AN    |
| state_in_ap    | in            | Andhra Pradesh                                           | AP    |
| state_in_ar    | in            | Arunachal Pradesh                                        | AR    |
| state_in_as    | in            | Assam                                                    | AS    |
| state_in_br    | in            | Bihar                                                    | BR    |
| state_in_ch    | in            | Chandigarh                                               | CH    |
| state_in_cg    | in            | Chattisgarh                                              | CG    |
| state_in_dn    | in            | Dadra and Nagar Haveli                                   | DN    |
| state_in_dd    | in            | Daman and Diu                                            | DD    |
| state_in_dl    | in            | Delhi                                                    | DL    |
| state_in_ga    | in            | Goa                                                      | GA    |
| state_in_gj    | in            | Gujarat                                                  | GJ    |
| state_in_hr    | in            | Haryana                                                  | HR    |
| state_in_hp    | in            | Himachal Pradesh                                         | HP    |
| state_in_jk    | in            | Jammu and Kashmir                                        | JK    |
| state_in_jh    | in            | Jharkhand                                                | JH    |
| state_in_ka    | in            | Karnataka                                                | KA    |
| state_in_kl    | in            | Kerala                                                   | KL    |
| state_in_ld    | in            | Lakshadweep                                              | LD    |
| state_in_mp    | in            | Madhya Pradesh                                           | MP    |
| state_in_mh    | in            | Maharashtra                                              | MH    |
| state_in_mn    | in            | Manipur                                                  | MN    |
| state_in_ml    | in            | Meghalaya                                                | ML    |
| state_in_mz    | in            | Mizoram                                                  | MZ    |
| state_in_nl    | in            | Nagaland                                                 | NL    |
| state_in_or    | in            | Orissa                                                   | OR    |
| state_in_py    | in            | Puducherry                                               | PY    |
| state_in_pb    | in            | Punjab                                                   | PB    |
| state_in_rj    | in            | Rajasthan                                                | RJ    |
| state_in_sk    | in            | Sikkim                                                   | SK    |
| state_in_tn    | in            | Tamil Nadu                                               | TN    |
| state_in_ts    | in            | Telangana                                                | TS    |
| state_in_tr    | in            | Tripura                                                  | TR    |
| state_in_up    | in            | Uttar Pradesh                                            | UP    |
| state_in_uk    | in            | Uttarakhand                                              | UK    |
| state_in_wb    | in            | West Bengal                                              | WB    |
| state_id_ac    | id            | Aceh                                                     | AC    |
| state_id_ba    | id            | Bali                                                     | BA    |
| state_id_bb    | id            | Bangka Belitung                                          | BB    |
| state_id_bt    | id            | Banten                                                   | BT    |
| state_id_be    | id            | Bengkulu                                                 | BE    |
| state_id_go    | id            | Gorontalo                                                | GO    |
| state_id_jk    | id            | Jakarta                                                  | JK    |
| state_id_ja    | id            | Jambi                                                    | JA    |
| state_id_jb    | id            | Jawa Barat                                               | JB    |
| state_id_jt    | id            | Jawa Tengah                                              | JT    |
| state_id_ji    | id            | Jawa Timur                                               | JI    |
| state_id_kb    | id            | Kalimantan Barat                                         | KB    |
| state_id_ks    | id            | Kalimantan Selatan                                       | KS    |
| state_id_kt    | id            | Kalimantan Tengah                                        | KT    |
| state_id_ki    | id            | Kalimantan Timur                                         | KI    |
| state_id_ku    | id            | Kalimantan Utara                                         | KU    |
| state_id_kr    | id            | Kepulauan Riau                                           | KR    |
| state_id_la    | id            | Lampung                                                  | LA    |
| state_id_ma    | id            | Maluku                                                   | MA    |
| state_id_mu    | id            | Maluku Utara                                             | MU    |
| state_id_nb    | id            | Nusa Tenggara Barat                                      | NB    |
| state_id_nt    | id            | Nusa Tenggara Timur                                      | NT    |
| state_id_pa    | id            | Papua                                                    | PA    |
| state_id_pb    | id            | Papua Barat                                              | PB    |
| state_id_ri    | id            | Riau                                                     | RI    |
| state_id_sr    | id            | Sulawesi Barat                                           | SR    |
| state_id_sn    | id            | Sulawesi Selatan                                         | SN    |
| state_id_st    | id            | Sulawesi Tengah                                          | ST    |
| state_id_sg    | id            | Sulawesi Tenggara                                        | SG    |
| state_id_sa    | id            | Sulawesi Utara                                           | SA    |
| state_id_sb    | id            | Sumatra Barat                                            | SB    |
| state_id_ss    | id            | Sumatra Selatan                                          | SS    |
| state_id_su    | id            | Sumatra Utara                                            | SU    |
| state_id_yo    | id            | Yogyakarta                                               | YO    |
| state_co_01    | co            | Antioquia                                                | ANT   |
| state_co_02    | co            | Atlántico                                                | ATL   |
| state_co_03    | co            | D.C.                                                     | DC    |
| state_co_04    | co            | Bolívar                                                  | BOL   |
| state_co_05    | co            | Boyacá                                                   | BOY   |
| state_co_06    | co            | Caldas                                                   | CAL   |
| state_co_07    | co            | Caquetá                                                  | CAQ   |
| state_co_08    | co            | Cauca                                                    | CAU   |
| state_co_09    | co            | Cesar                                                    | CES   |
| state_co_10    | co            | Córdoba                                                  | COR   |
| state_co_11    | co            | Cundinamarca                                             | CUN   |
| state_co_12    | co            | Chocó                                                    | CHO   |
| state_co_13    | co            | Huila                                                    | HUI   |
| state_co_14    | co            | La Guajira                                               | LAG   |
| state_co_15    | co            | Magdalena                                                | MAG   |
| state_co_16    | co            | Meta                                                     | MET   |
| state_co_17    | co            | Nariño                                                   | NAR   |
| state_co_18    | co            | Norte de Santander                                       | NSA   |
| state_co_19    | co            | Quindio                                                  | QUI   |
| state_co_20    | co            | Risaralda                                                | RIS   |
| state_co_21    | co            | Santander                                                | SAN   |
| state_co_22    | co            | Sucre                                                    | SUC   |
| state_co_23    | co            | Tolima                                                   | TOL   |
| state_co_24    | co            | Valle del Cauca                                          | VAC   |
| state_co_25    | co            | Arauca                                                   | ARA   |
| state_co_26    | co            | Casanare                                                 | CAS   |
| state_co_27    | co            | Putumayo                                                 | PUT   |
| state_co_28    | co            | Archipiélago de San Andrés, Providencia y Santa Catalina | SAP   |
| state_co_29    | co            | Amazonas                                                 | AMA   |
| state_co_30    | co            | Guainía                                                  | GUA   |
| state_co_31    | co            | Guaviare                                                 | GUV   |
| state_co_32    | co            | Vaupés                                                   | VAU   |
| state_co_33    | co            | Vichada                                                  | VID   |
| state_mn_01    | mn            | Архангай                                                 | 01    |
| state_mn_02    | mn            | Баян-Өлгий                                               | 02    |
| state_mn_03    | mn            | Баянхонгор                                               | 03    |
| state_mn_04    | mn            | Булган                                                   | 04    |
| state_mn_05    | mn            | Говь-Алтай                                               | 05    |
| state_mn_06    | mn            | Дорноговь                                                | 06    |
| state_mn_07    | mn            | Дорнод                                                   | 07    |
| state_mn_08    | mn            | Дундговь                                                 | 08    |
| state_mn_09    | mn            | Завхан                                                   | 09    |
| state_mn_10    | mn            | Өвөрхангай                                               | 10    |
| state_mn_11    | mn            | Өмнөговь                                                 | 11    |
| state_mn_12    | mn            | Сүхбаатар                                                | 12    |
| state_mn_13    | mn            | Сэлэнгэ                                                  | 13    |
| state_mn_14    | mn            | Төв                                                      | 14    |
| state_mn_15    | mn            | Увс                                                      | 15    |
| state_mn_16    | mn            | Ховд                                                     | 16    |
| state_mn_17    | mn            | Хөвсгөл                                                  | 17    |
| state_mn_18    | mn            | Хэнтий                                                   | 18    |
| state_mn_19    | mn            | Дархан-Уул                                               | 19    |
| state_mn_20    | mn            | Орхон                                                    | 20    |
| state_mn_23    | mn            | УБ - Хан Уул                                             | 23    |
| state_mn_24    | mn            | УБ - Баянзүрх                                            | 24    |
| state_mn_25    | mn            | УБ - Сүхбаатар                                           | 25    |
| state_mn_26    | mn            | УБ - Баянгол                                             | 26    |
| state_mn_27    | mn            | УБ - Багануур                                            | 27    |
| state_mn_28    | mn            | УБ - Багахангай                                          | 28    |
| state_mn_29    | mn            | УБ - Налайх                                              | 29    |
| state_mn_32    | mn            | Говьсүмбэр                                               | 32    |
| state_mn_34    | mn            | УБ - Сонгино Хайрхан                                     | 34    |
| state_mn_35    | mn            | УБ - Чингэлтэй                                           | 35    |
| state_uk1      | uk            | Aberdeenshire                                            | A1    |
| state_uk2      | uk            | Angus                                                    | A5    |
| state_uk3      | uk            | Argyll                                                   | A7    |
| state_uk4      | uk            | Avon                                                     | A9    |
| state_uk5      | uk            | Ayrshire                                                 | B1    |
| state_uk6      | uk            | Banffshire                                               | B3    |
| state_uk7      | uk            | Bedfordshire                                             | B5    |
| state_uk8      | uk            | Berkshire                                                | B7    |
| state_uk9      | uk            | Berwickshire                                             | B9    |
| state_uk10     | uk            | Buckinghamshire                                          | C1    |
| state_uk11     | uk            | Caithness                                                | C3    |
| state_uk12     | uk            | Cambridgeshire                                           | C5    |
| state_uk13     | uk            | Channel Islands                                          | C6    |
| state_uk14     | uk            | Cheshire                                                 | C7    |
| state_uk15     | uk            | Clackmannanshire                                         | C9    |
| state_uk16     | uk            | Cleveland                                                | D1    |
| state_uk17     | uk            | Clwyd                                                    | D3    |
| state_uk18     | uk            | County Antrim                                            | D5    |
| state_uk19     | uk            | County Armagh                                            | D7    |
| state_uk20     | uk            | County Down                                              | D9    |
| state_uk21     | uk            | County Durham                                            | E1    |
| state_uk22     | uk            | County Fermanagh                                         | E3    |
| state_uk23     | uk            | County Londonderry                                       | E5    |
| state_uk24     | uk            | County Tyrone                                            | E7    |
| state_uk25     | uk            | Cornwall                                                 | E9    |
| state_uk26     | uk            | Cumbria                                                  | F1    |
| state_uk27     | uk            | Derbyshire                                               | F3    |
| state_uk28     | uk            | Devon                                                    | F5    |
| state_uk29     | uk            | Dorset                                                   | F7    |
| state_uk30     | uk            | Dumfriesshire                                            | F9    |
| state_uk31     | uk            | Dunbartonshire                                           | G1    |
| state_uk32     | uk            | Dyfed                                                    | G3    |
| state_uk33     | uk            | East Lothian                                             | G5    |
| state_uk34     | uk            | East Sussex                                              | G7    |
| state_uk35     | uk            | Essex                                                    | G9    |
| state_uk36     | uk            | Fife                                                     | H1    |
| state_uk37     | uk            | Gloucestershire                                          | H3    |
| state_uk38     | uk            | Gwent                                                    | H7    |
| state_uk39     | uk            | Gwynedd                                                  | H9    |
| state_uk40     | uk            | Hampshire                                                | I1    |
| state_uk41     | uk            | Herefordshire                                            | I3    |
| state_uk42     | uk            | Hertfordshire                                            | I5    |
| state_uk43     | uk            | Inverness-Shire                                          | I7    |
| state_uk44     | uk            | Isle of Arran                                            | I9    |
| state_uk45     | uk            | Isle of Barra                                            | J1    |
| state_uk46     | uk            | Isle of Benbecula                                        | J3    |
| state_uk47     | uk            | Isle of Bute                                             | J5    |
| state_uk48     | uk            | Isle of Canna                                            | J7    |
| state_uk49     | uk            | Isle of Coll                                             | J9    |
| state_uk50     | uk            | Isle of Colonsay                                         | K1    |
| state_uk51     | uk            | Isle of Cumbrae                                          | K3    |
| state_uk52     | uk            | Isle of Eigg                                             | K5    |
| state_uk53     | uk            | Isle of Gigha                                            | K7    |
| state_uk54     | uk            | Isle of Harris                                           | K9    |
| state_uk55     | uk            | Isle of Iona                                             | L1    |
| state_uk56     | uk            | Isle of Islay                                            | L2    |
| state_uk57     | uk            | Isle of Jura                                             | L5    |
| state_uk58     | uk            | Isle of Lewis                                            | L7    |
| state_uk59     | uk            | Isle of Man                                              | L9    |
| state_uk60     | uk            | Isle of Mull                                             | M1    |
| state_uk61     | uk            | Isle of North Uist                                       | M3    |
| state_uk62     | uk            | Isle of Rhum                                             | M7    |
| state_uk63     | uk            | Isle of Scalpay                                          | M9    |
| state_uk64     | uk            | Shetland Islands                                         | N1    |
| state_uk65     | uk            | Isle of Skye                                             | N3    |
| state_uk66     | uk            | Isle of South Uist                                       | N5    |
| state_uk67     | uk            | Isle of Tiree                                            | N7    |
| state_uk68     | uk            | Isle of Wight                                            | N9    |
| state_uk69     | uk            | Kent                                                     | O5    |
| state_uk70     | uk            | Kincardineshire                                          | O7    |
| state_uk71     | uk            | Kinross-Shire                                            | O9    |
| state_uk72     | uk            | Kirkcudbrightshire                                       | P1    |
| state_uk73     | uk            | Lancashire                                               | P5    |
| state_uk74     | uk            | Leicestershire                                           | P7    |
| state_uk75     | uk            | Lincolnshire                                             | P9    |
| state_uk76     | uk            | Merseyside                                               | Q3    |
| state_uk77     | uk            | Mid Glamorgan                                            | Q5    |
| state_uk78     | uk            | Middlesex                                                | Q9    |
| state_uk79     | uk            | Morayshire                                               | R1    |
| state_uk80     | uk            | Nairnshire                                               | R3    |
| state_uk81     | uk            | North Humberside                                         | R7    |
| state_uk82     | uk            | North Yorkshire                                          | R9    |
| state_uk83     | uk            | Northamptonshire                                         | S1    |
| state_uk84     | uk            | Northumberland                                           | S3    |
| state_uk85     | uk            | Nottinghamshire                                          | S5    |
| state_uk86     | uk            | Oxfordshire                                              | S7    |
| state_uk87     | uk            | Peeblesshire                                             | S9    |
| state_uk88     | uk            | Perthshire                                               | T1    |
| state_uk89     | uk            | Powys                                                    | T3    |
| state_uk90     | uk            | Renfrewshire                                             | T5    |
| state_uk91     | uk            | Ross-Shire                                               | T7    |
| state_uk92     | uk            | Roxburghshire                                            | T9    |
| state_uk93     | uk            | Selkirkshire                                             | U3    |
| state_uk94     | uk            | Shropshire                                               | U5    |
| state_uk95     | uk            | Somerset                                                 | U7    |
| state_uk96     | uk            | South Glamorgan                                          | U9    |
| state_uk97     | uk            | South Humberside                                         | V1    |
| state_uk98     | uk            | South Yorkshire                                          | V3    |
| state_uk99     | uk            | Staffordshire                                            | V5    |
| state_uk100    | uk            | Stirlingshire                                            | V7    |
| state_uk101    | uk            | Suffolk                                                  | V9    |
| state_uk102    | uk            | Surrey                                                   | W1    |
| state_uk103    | uk            | Sutherland                                               | W3    |
| state_uk104    | uk            | Tyne and Wear                                            | W5    |
| state_uk105    | uk            | Warwickshire                                             | W7    |
| state_uk106    | uk            | West Glamorgan                                           | W9    |
| state_uk107    | uk            | West Lothian                                             | X1    |
| state_uk108    | uk            | West Midlands                                            | X3    |
| state_uk109    | uk            | West Sussex                                              | X5    |
| state_uk110    | uk            | West Yorkshire                                           | X7    |
| state_uk111    | uk            | Wigtownshire                                             | X9    |
| state_uk112    | uk            | Wiltshire                                                | Y1    |
| state_uk113    | uk            | Worcestershire                                           | Y3    |
| state_uk114    | uk            | Orkney                                                   | M5    |
| state_uk115    | uk            | Isles of Scilly                                          | O1    |
| state_uk116    | uk            | Lanarkshire                                              | P3    |
| state_uk117    | uk            | London                                                   | Q1    |
| state_uk118    | uk            | Midlothian                                               | Q7    |
| state_uk119    | uk            | Norfolk                                                  | R5    |
| RO_AB          | ro            | Alba                                                     | AB    |
| RO_AG          | ro            | Argeș                                                    | AG    |
| RO_AR          | ro            | Arad                                                     | AR    |
| RO_B           | ro            | București                                                | B     |
| RO_BC          | ro            | Bacău                                                    | BC    |
| RO_BH          | ro            | Bihor                                                    | BH    |
| RO_BN          | ro            | Bistrița-Năsăud                                          | BN    |
| RO_BR          | ro            | Brăila                                                   | BR    |
| RO_BT          | ro            | Botoșani                                                 | BT    |
| RO_BV          | ro            | Brașov                                                   | BV    |
| RO_BZ          | ro            | Buzău                                                    | BZ    |
| RO_CJ          | ro            | Cluj                                                     | CJ    |
| RO_CL          | ro            | Călărași                                                 | CL    |
| RO_CS          | ro            | Caraș Severin                                            | CS    |
| RO_CT          | ro            | Constanța                                                | CT    |
| RO_CV          | ro            | Covasna                                                  | CV    |
| RO_DB          | ro            | Dâmbovița                                                | DB    |
| RO_DJ          | ro            | Dolj                                                     | DJ    |
| RO_GJ          | ro            | Gorj                                                     | GJ    |
| RO_GL          | ro            | Galați                                                   | GL    |
| RO_GR          | ro            | Giurgiu                                                  | GR    |
| RO_HD          | ro            | Hunedoara                                                | HD    |
| RO_HR          | ro            | Harghita                                                 | HR    |
| RO_IF          | ro            | Ilfov                                                    | IF    |
| RO_IL          | ro            | Ialomița                                                 | IL    |
| RO_IS          | ro            | Iași                                                     | IS    |
| RO_MH          | ro            | Mehedinți                                                | MH    |
| RO_MM          | ro            | Maramureș                                                | MM    |
| RO_MS          | ro            | Mureș                                                    | MS    |
| RO_NT          | ro            | Neamț                                                    | NT    |
| RO_OT          | ro            | Olt                                                      | OT    |
| RO_PH          | ro            | Prahova                                                  | PH    |
| RO_SB          | ro            | Sibiu                                                    | SB    |
| RO_SJ          | ro            | Sălaj                                                    | SJ    |
| RO_SM          | ro            | Satu Mare                                                | SM    |
| RO_SV          | ro            | Suceava                                                  | SV    |
| RO_TL          | ro            | Tulcea                                                   | TL    |
| RO_TM          | ro            | Timiș                                                    | TM    |
| RO_TR          | ro            | Teleorman                                                | TR    |
| RO_VL          | ro            | Vâlcea                                                   | VL    |
| RO_VN          | ro            | Vrancea                                                  | VN    |
| RO_VS          | ro            | Vaslui                                                   | VS    |
| state_cn_BJ    | cn            | 北京市                                                   | 京    |
| state_cn_SH    | cn            | 上海市                                                   | 沪    |
| state_cn_ZJ    | cn            | 浙江省                                                   | 浙    |
| state_cn_TJ    | cn            | 天津市                                                   | 津    |
| state_cn_AH    | cn            | 安徽省                                                   | 皖    |
| state_cn_FJ    | cn            | 福建省                                                   | 闽    |
| state_cn_CQ    | cn            | 重庆市                                                   | 渝    |
| state_cn_JX    | cn            | 江西省                                                   | 赣    |
| state_cn_SD    | cn            | 山东省                                                   | 鲁    |
| state_cn_HA    | cn            | 河南省                                                   | 豫    |
| state_cn_NM    | cn            | 内蒙古自治区                                             | 蒙    |
| state_cn_HB    | cn            | 湖北省                                                   | 鄂    |
| state_cn_XJ    | cn            | 新疆维吾尔自治区                                         | 新    |
| state_cn_HN    | cn            | 湖南省                                                   | 湘    |
| state_cn_NX    | cn            | 宁夏回族自治区                                           | 宁    |
| state_cn_GD    | cn            | 广东省                                                   | 粤    |
| state_cn_XZ    | cn            | 西藏自治区                                               | 藏    |
| state_cn_HI    | cn            | 海南省                                                   | 琼    |
| state_cn_GX    | cn            | 广西壮族自治区                                           | 桂    |
| state_cn_SC    | cn            | 四川省                                                   | 蜀    |
| state_cn_HE    | cn            | 河北省                                                   | 冀    |
| state_cn_GZ    | cn            | 贵州省                                                   | 黔    |
| state_cn_SX    | cn            | 山西省                                                   | 晋    |
| state_cn_YN    | cn            | 云南省                                                   | 滇    |
| state_cn_LN    | cn            | 辽宁省                                                   | 辽    |
| state_cn_SN    | cn            | 陕西省                                                   | 陕    |
| state_cn_JL    | cn            | 吉林省                                                   | 吉    |
| state_cn_GS    | cn            | 甘肃省                                                   | 甘    |
| state_cn_HL    | cn            | 黑龙江省                                                 | 黑    |
| state_cn_QH    | cn            | 青海省                                                   | 青    |
| state_cn_JS    | cn            | 江苏省                                                   | 苏    |
| state_cn_TW    | cn            | 台湾省                                                   | 台    |
| state_cn_HK    | cn            | 香港特别行政区                                           | 港    |
| state_cn_MO    | cn            | 澳门特别行政区                                           | 澳    |
| state_et_1     | et            | Addis Ababa                                              | AA    |
| state_et_2     | et            | Afar                                                     | AF    |
| state_et_3     | et            | Amhara                                                   | AM    |
| state_et_4     | et            | Benishangul-Gumuz                                        | BN    |
| state_et_5     | et            | Dire Dawa                                                | DR    |
| state_et_6     | et            | Gambella Peoples                                         | GM    |
| state_et_7     | et            | Harrari Peoples                                          | HR    |
| state_et_8     | et            | Oromia                                                   | OR    |
| state_et_9     | et            | Somalia                                                  | SM    |
| state_et_10    | et            | Southern Peoples, Nations, and Nationalities             | SP    |
| state_et_11    | et            | Tigray                                                   | TG    |
| state_ie_1     | ie            | Carlow                                                   | CW    |
| state_ie_2     | ie            | Cavan                                                    | CN    |
| state_ie_3     | ie            | Clare                                                    | CE    |
| state_ie_4     | ie            | Cork                                                     | C     |
| state_ie_5     | ie            | Limerick                                                 | LK    |
| state_ie_6     | ie            | Waterford                                                | WD    |
| state_ie_7     | ie            | Donegal                                                  | DL    |
| state_ie_8     | ie            | Dublin                                                   | D     |
| state_ie_9     | ie            | Galway                                                   | G     |
| state_ie_10    | ie            | Kerry                                                    | KY    |
| state_ie_11    | ie            | Kildare                                                  | KE    |
| state_ie_12    | ie            | Kilkenny                                                 | KK    |
| state_ie_13    | ie            | Laois                                                    | LS    |
| state_ie_14    | ie            | Leitrim                                                  | LM    |
| state_ie_15    | ie            | Longford                                                 | LD    |
| state_ie_16    | ie            | Louth                                                    | LH    |
| state_ie_17    | ie            | Mayo                                                     | MO    |
| state_ie_18    | ie            | Meath                                                    | MH    |
| state_ie_19    | ie            | Monaghan                                                 | MN    |
| state_ie_20    | ie            | Offaly                                                   | OY    |
| state_ie_21    | ie            | Roscommon                                                | RN    |
| state_ie_22    | ie            | Sligo                                                    | SO    |
| state_ie_23    | ie            | Tipperary                                                | TR    |
| state_ie_24    | ie            | Westmeath                                                | WH    |
| state_ie_25    | ie            | Wexford                                                  | WX    |
| state_ie_26    | ie            | Wicklow                                                  | WW    |
| state_ie_27    | ie            | Antrim                                                   | AM    |
| state_ie_28    | ie            | Armagh                                                   | AH    |
| state_ie_29    | ie            | Down                                                     | DN    |
| state_ie_30    | ie            | Fermanagh                                                | FH    |
| state_ie_31    | ie            | Londonderry                                              | LY    |
| state_ie_32    | ie            | Tyrone                                                   | TE    |
| state_nl_dr    | nl            | Drenthe                                                  | DR    |
| state_nl_fl    | nl            | Flevoland                                                | FL    |
| state_nl_fr    | nl            | Friesland                                                | FR    |
| state_nl_ge    | nl            | Gelderland                                               | GE    |
| state_nl_gr    | nl            | Groningen                                                | GR    |
| state_nl_li    | nl            | Limburg                                                  | LI    |
| state_nl_nb    | nl            | Noord-Brabant                                            | NB    |
| state_nl_nh    | nl            | Noord-Holland                                            | NH    |
| state_nl_ov    | nl            | Overijssel                                               | OV    |
| state_nl_ut    | nl            | Utrecht                                                  | UT    |
| state_nl_ze    | nl            | Zeeland                                                  | ZE    |
| state_nl_zh    | nl            | Zuid-Holland                                             | ZH    |
| state_nl_bq1   | nl            | Bonaire                                                  | BQ1   |
| state_nl_bq2   | nl            | Saba                                                     | BQ2   |
| state_nl_bq3   | nl            | Sint Eustatius                                           | BQ3   |
| state_tr_01    | tr            | Adana                                                    | 01    |
| state_tr_02    | tr            | Adıyaman                                                 | 02    |
| state_tr_03    | tr            | Afyon                                                    | 03    |
| state_tr_04    | tr            | Ağrı                                                     | 04    |
| state_tr_05    | tr            | Amasya                                                   | 05    |
| state_tr_06    | tr            | Ankara                                                   | 06    |
| state_tr_07    | tr            | Antalya                                                  | 07    |
| state_tr_08    | tr            | Artvin                                                   | 08    |
| state_tr_09    | tr            | Aydın                                                    | 09    |
| state_tr_10    | tr            | Balıkesir                                                | 10    |
| state_tr_11    | tr            | Bilecik                                                  | 11    |
| state_tr_12    | tr            | Bingöl                                                   | 12    |
| state_tr_13    | tr            | Bitlis                                                   | 13    |
| state_tr_14    | tr            | Bolu                                                     | 14    |
| state_tr_15    | tr            | Burdur                                                   | 15    |
| state_tr_16    | tr            | Bursa                                                    | 16    |
| state_tr_17    | tr            | Çanakkale                                                | 17    |
| state_tr_18    | tr            | Çankırı                                                  | 18    |
| state_tr_19    | tr            | Çorum                                                    | 19    |
| state_tr_20    | tr            | Denizli                                                  | 20    |
| state_tr_21    | tr            | Diyarbakır                                               | 21    |
| state_tr_22    | tr            | Edirne                                                   | 22    |
| state_tr_23    | tr            | Elazığ                                                   | 23    |
| state_tr_24    | tr            | Erzincan                                                 | 24    |
| state_tr_25    | tr            | Erzurum                                                  | 25    |
| state_tr_26    | tr            | Eskişehir                                                | 26    |
| state_tr_27    | tr            | Gaziantep                                                | 27    |
| state_tr_28    | tr            | Giresun                                                  | 28    |
| state_tr_29    | tr            | Gümüşhane                                                | 29    |
| state_tr_30    | tr            | Hakkari                                                  | 30    |
| state_tr_31    | tr            | Hatay                                                    | 31    |
| state_tr_32    | tr            | Isparta                                                  | 32    |
| state_tr_33    | tr            | İçel                                                     | 33    |
| state_tr_34    | tr            | İstanbul                                                 | 34    |
| state_tr_35    | tr            | İzmir                                                    | 35    |
| state_tr_36    | tr            | Kars                                                     | 36    |
| state_tr_37    | tr            | Kastamonu                                                | 37    |
| state_tr_38    | tr            | Kayseri                                                  | 38    |
| state_tr_39    | tr            | Kırklareli                                               | 39    |
| state_tr_40    | tr            | Kırşehir                                                 | 40    |
| state_tr_41    | tr            | Kocaeli                                                  | 41    |
| state_tr_42    | tr            | Konya                                                    | 42    |
| state_tr_43    | tr            | Kütahya                                                  | 43    |
| state_tr_44    | tr            | Malatya                                                  | 44    |
| state_tr_45    | tr            | Manisa                                                   | 45    |
| state_tr_46    | tr            | K.maraş                                                  | 46    |
| state_tr_47    | tr            | Mardin                                                   | 47    |
| state_tr_48    | tr            | Muğla                                                    | 48    |
| state_tr_49    | tr            | Muş                                                      | 49    |
| state_tr_50    | tr            | Nevşehir                                                 | 50    |
| state_tr_51    | tr            | Niğde                                                    | 51    |
| state_tr_52    | tr            | Ordu                                                     | 52    |
| state_tr_53    | tr            | Rize                                                     | 53    |
| state_tr_54    | tr            | Sakarya                                                  | 54    |
| state_tr_55    | tr            | Samsun                                                   | 55    |
| state_tr_56    | tr            | Siirt                                                    | 56    |
| state_tr_57    | tr            | Sinop                                                    | 57    |
| state_tr_58    | tr            | Sivas                                                    | 58    |
| state_tr_59    | tr            | Tekirdağ                                                 | 59    |
| state_tr_60    | tr            | Tokat                                                    | 60    |
| state_tr_61    | tr            | Trabzon                                                  | 61    |
| state_tr_62    | tr            | Tunceli                                                  | 62    |
| state_tr_63    | tr            | Şanlıurfa                                                | 63    |
| state_tr_64    | tr            | Uşak                                                     | 64    |
| state_tr_65    | tr            | Van                                                      | 65    |
| state_tr_66    | tr            | Yozgat                                                   | 66    |
| state_tr_67    | tr            | Zonguldak                                                | 67    |
| state_tr_68    | tr            | Aksaray                                                  | 68    |
| state_tr_69    | tr            | Bayburt                                                  | 69    |
| state_tr_70    | tr            | Karaman                                                  | 70    |
| state_tr_71    | tr            | Kırıkkale                                                | 71    |
| state_tr_72    | tr            | Batman                                                   | 72    |
| state_tr_73    | tr            | Şırnak                                                   | 73    |
| state_tr_74    | tr            | Bartın                                                   | 74    |
| state_tr_75    | tr            | Ardahan                                                  | 75    |
| state_tr_76    | tr            | Iğdır                                                    | 76    |
| state_tr_77    | tr            | Yalova                                                   | 77    |
| state_tr_78    | tr            | Karabük                                                  | 78    |
| state_tr_79    | tr            | Kilis                                                    | 79    |
| state_tr_80    | tr            | Osmaniye                                                 | 80    |
| state_tr_81    | tr            | Düzce                                                    | 81    |
| state_vn_VN-44 | vn            | An Giang                                                 | VN-44 |
| state_vn_VN-57 | vn            | Bình Dương                                               | VN-57 |
| state_vn_VN-31 | vn            | Bình Định                                                | VN-31 |
| state_vn_VN-54 | vn            | Bắc Giang                                                | VN-54 |
| state_vn_VN-53 | vn            | Bắc Kạn                                                  | VN-53 |
| state_vn_VN-55 | vn            | Bạc Liêu                                                 | VN-55 |
| state_vn_VN-56 | vn            | Bắc Ninh                                                 | VN-56 |
| state_vn_VN-58 | vn            | Bình Phước                                               | VN-58 |
| state_vn_VN-43 | vn            | Bà Rịa - Vũng Tàu                                        | VN-43 |
| state_vn_VN-40 | vn            | Bình Thuận                                               | VN-40 |
| state_vn_VN-50 | vn            | Bến Tre                                                  | VN-50 |
| state_vn_VN-04 | vn            | Cao Bằng                                                 | VN-04 |
| state_vn_VN-59 | vn            | Cà Mau                                                   | VN-59 |
| state_vn_VN-CT | vn            | TP Cần Thơ                                               | VN-CT |
| state_vn_VN-71 | vn            | Điện Biên                                                | VN-71 |
| state_vn_VN-33 | vn            | Đắk Lắk                                                  | VN-33 |
| state_vn_VN-DN | vn            | TP Đà Nẵng                                               | VN-DN |
| state_vn_VN-39 | vn            | Đồng Nai                                                 | VN-39 |
| state_vn_VN-72 | vn            | Đắk Nông                                                 | VN-72 |
| state_vn_VN-45 | vn            | Đồng Tháp                                                | VN-45 |
| state_vn_VN-30 | vn            | Gia Lai                                                  | VN-30 |
| state_vn_VN-14 | vn            | Hòa Bình                                                 | VN-14 |
| state_vn_VN-SG | vn            | TP Hồ Chí Minh                                           | VN-SG |
| state_vn_VN-61 | vn            | Hải Dương                                                | VN-61 |
| state_vn_VN-73 | vn            | Hậu Giang                                                | VN-73 |
| state_vn_VN-03 | vn            | Hà Giang                                                 | VN-03 |
| state_vn_VN-HN | vn            | Hà Nội                                                   | VN-HN |
| state_vn_VN-63 | vn            | Hà Nam                                                   | VN-63 |
| state_vn_VN-HP | vn            | TP Hải Phòng                                             | VN-HP |
| state_vn_VN-23 | vn            | Hà Tĩnh                                                  | VN-23 |
| state_vn_VN-66 | vn            | Hưng Yên                                                 | VN-66 |
| state_vn_VN-47 | vn            | Kiên Giang                                               | VN-47 |
| state_vn_VN-34 | vn            | Khánh Hòa                                                | VN-34 |
| state_vn_VN-28 | vn            | Kon Tum                                                  | VN-28 |
| state_vn_VN-41 | vn            | Long An                                                  | VN-41 |
| state_vn_VN-02 | vn            | Lào Cai                                                  | VN-02 |
| state_vn_VN-01 | vn            | Lai Châu                                                 | VN-01 |
| state_vn_VN-35 | vn            | Lâm Đồng                                                 | VN-35 |
| state_vn_VN-09 | vn            | Lạng Sơn                                                 | VN-09 |
| state_vn_VN-22 | vn            | Nghệ An                                                  | VN-22 |
| state_vn_VN-18 | vn            | Ninh Bình                                                | VN-18 |
| state_vn_VN-67 | vn            | Nam Định                                                 | VN-67 |
| state_vn_VN-36 | vn            | Ninh Thuận                                               | VN-36 |
| state_vn_VN-68 | vn            | Phú Thọ                                                  | VN-68 |
| state_vn_VN-32 | vn            | Phú Yên                                                  | VN-32 |
| state_vn_VN-24 | vn            | Quảng Bình                                               | VN-24 |
| state_vn_VN-13 | vn            | Quảng Ninh                                               | VN-13 |
| state_vn_VN-27 | vn            | Quảng Nam                                                | VN-27 |
| state_vn_VN-29 | vn            | Quảng Ngãi                                               | VN-29 |
| state_vn_VN-25 | vn            | Quảng Trị                                                | VN-25 |
| state_vn_VN-05 | vn            | Sơn La                                                   | VN-05 |
| state_vn_VN-52 | vn            | Sóc Trăng                                                | VN-52 |
| state_vn_VN-20 | vn            | Thái Bình                                                | VN-20 |
| state_vn_VN-46 | vn            | Tiền Giang                                               | VN-46 |
| state_vn_VN-21 | vn            | Thanh Hóa                                                | VN-21 |
| state_vn_VN-69 | vn            | Thái Nguyên                                              | VN-69 |
| state_vn_VN-37 | vn            | Tây Ninh                                                 | VN-37 |
| state_vn_VN-07 | vn            | Tuyên Quang                                              | VN-07 |
| state_vn_VN-26 | vn            | Thừa Thiên - Huế                                         | VN-26 |
| state_vn_VN-51 | vn            | Trà Vinh                                                 | VN-51 |
| state_vn_VN-49 | vn            | Vĩnh Long                                                | VN-49 |
| state_vn_VN-70 | vn            | Vĩnh Phúc                                                | VN-70 |
| state_vn_VN-06 | vn            | Yên Bái                                                  | VN-06 |
| state_SJ       | cr            | San José                                                 | CR-SJ |
| state_A        | cr            | Alajuela                                                 | CR-A  |
| state_H        | cr            | Heredia                                                  | CR-H  |
| state_C        | cr            | Cartago                                                  | CR-C  |
| state_P        | cr            | Puntarenas                                               | CR-P  |
| state_G        | cr            | Guanacaste                                               | CR-G  |
| state_L        | cr            | Limón                                                    | CR-L  |
| state_DO_01    | do            | Distrito Nacional                                        | DN    |
| state_DO_02    | do            | Azua                                                     | AZU   |
| state_DO_03    | do            | Bahoruco                                                 | BAH   |
| state_DO_04    | do            | Barahona                                                 | BAR   |
| state_DO_05    | do            | Dajabón                                                  | DAJ   |
| state_DO_06    | do            | Duarte                                                   | DUA   |
| state_DO_07    | do            | Elías Piña                                               | ELP   |
| state_DO_08    | do            | El Seibo                                                 | ELS   |
| state_DO_09    | do            | Espaillat                                                | ESP   |
| state_DO_10    | do            | Independencia                                            | IND   |
| state_DO_11    | do            | La Altagracia                                            | LA    |
| state_DO_12    | do            | La Romana                                                | LR    |
| state_DO_13    | do            | La Vega                                                  | LV    |
| state_DO_14    | do            | María Trinidad Sánchez                                   | MTS   |
| state_DO_15    | do            | Monte Cristi                                             | MC    |
| state_DO_16    | do            | Pedernales                                               | PED   |
| state_DO_17    | do            | Peravia                                                  | PER   |
| state_DO_18    | do            | Puerto Plata                                             | PP    |
| state_DO_19    | do            | Hermanas Mirabal                                         | HEM   |
| state_DO_20    | do            | Samaná                                                   | SAM   |
| state_DO_21    | do            | San Cristóbal                                            | SC    |
| state_DO_22    | do            | San Juan                                                 | SJ    |
| state_DO_23    | do            | San Pedro de Macorís                                     | SPM   |
| state_DO_24    | do            | Sánchez Ramírez                                          | SRA   |
| state_DO_25    | do            | Santiago                                                 | STGO  |
| state_DO_26    | do            | Santiago Rodríguez                                       | SRO   |
| state_DO_27    | do            | Valverde                                                 | VAL   |
| state_DO_28    | do            | Monseñor Nouel                                           | MON   |
| state_DO_29    | do            | Monte Plata                                              | MP    |
| state_DO_30    | do            | Hato Mayor                                               | HAM   |
| state_DO_31    | do            | San José de Ocoa                                         | SJO   |
| state_DO_32    | do            | Santo Domingo                                            | SD    |
| state_pe_01    | pe            | Amazonas                                                 | 01    |
| state_pe_02    | pe            | Áncash                                                   | 02    |
| state_pe_03    | pe            | Apurímac                                                 | 03    |
| state_pe_04    | pe            | Arequipa                                                 | 04    |
| state_pe_05    | pe            | Ayacucho                                                 | 05    |
| state_pe_06    | pe            | Cajamarca                                                | 06    |
| state_pe_07    | pe            | Callao                                                   | 07    |
| state_pe_08    | pe            | Cusco                                                    | 08    |
| state_pe_09    | pe            | Huancavelica                                             | 09    |
| state_pe_10    | pe            | Huánuco                                                  | 10    |
| state_pe_11    | pe            | Ica                                                      | 11    |
| state_pe_12    | pe            | Junin                                                    | 12    |
| state_pe_13    | pe            | La Libertad                                              | 13    |
| state_pe_14    | pe            | Lambayeque                                               | 14    |
| state_pe_15    | pe            | Lima                                                     | 15    |
| state_pe_16    | pe            | Loreto                                                   | 16    |
| state_pe_17    | pe            | Madre de Dios                                            | 17    |
| state_pe_18    | pe            | Moquegua                                                 | 18    |
| state_pe_19    | pe            | Pasco                                                    | 19    |
| state_pe_20    | pe            | Piura                                                    | 20    |
| state_pe_21    | pe            | Puno                                                     | 21    |
| state_pe_22    | pe            | San Martín                                               | 22    |
| state_pe_23    | pe            | Tacna                                                    | 23    |
| state_pe_24    | pe            | Tumbes                                                   | 24    |
| state_pe_25    | pe            | Ucayali                                                  | 25    |
| state_cl_01    | cl            | Tarapacá                                                 | 01    |
| state_cl_02    | cl            | Antofagasta                                              | 02    |
| state_cl_03    | cl            | Atacama                                                  | 03    |
| state_cl_04    | cl            | Coquimbo                                                 | 04    |
| state_cl_05    | cl            | Valparaíso                                               | 05    |
| state_cl_06    | cl            | del Libertador Gral. Bernardo O’Higgins                  | 06    |
| state_cl_07    | cl            | del Maule                                                | 07    |
| state_cl_08    | cl            | del BíoBio                                               | 08    |
| state_cl_09    | cl            | de la Araucania                                          | 09    |
| state_cl_10    | cl            | de los Lagos                                             | 10    |
| state_cl_11    | cl            | Aysén del Gral. Carlos Ibáñez del Campo                  | 11    |
| state_cl_12    | cl            | Magallanes                                               | 12    |
| state_cl_13    | cl            | Metropolitana                                            | 13    |
| state_cl_14    | cl            | Los Ríos                                                 | 14    |
| state_cl_15    | cl            | Arica y Parinacota                                       | 15    |
| state_cl_16    | cl            | del Ñuble                                                | 16    |

对于每行（记录）：

- 第一列是要创建或更新的记录的 [外部id](https://www.odoo.com/documentation/13.0/glossary.html#term-external-id)
- 第二列是要关联的国家对象的[外部id](https://www.odoo.com/documentation/13.0/glossary.html#term-external-id) (国家对象必须提前定义)
- 第三列是针对`res.country.state`的 `name` 字段
- 第四列是针对`res.country.state`的`code` 字段