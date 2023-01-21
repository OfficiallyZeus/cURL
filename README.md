https://everything.curl.com


{curl 8.0 apple}
{curl-file-attributes character-encoding = "utf-16"}
{import * from CURL.DESKTOP.CLIPBOARD}
{def sql-ta:TextArea =
    {TextArea
        ||width = {make-elastic preferred-size = 1in},
        height = {make-elastic preferred-size = 1in}
    }
}


{def param-ta:TextArea =
    {TextArea
        ||width = 7in,
        height = {make-elastic preferred-size = 0.5in}
    }
}

{def comp-ta:TextArea =
    {TextArea
        ||width = 7in,
        height = {make-elastic preferred-size = 0.5in}
    }
}


{define-proc {get-type-list
                 index:int,
                 val:String,
                 params:StringArray
             }:DropdownList

    || ■CHAR
    || ■NUMBER
    || ■VARCHAR
    || ■TIMESTAMP
    || ■DATE

    def dl =  {DropdownList
                  name = val,
                  user-data = index,
                  "文字列",
                  "数値",
                  {on ValueChanged at dl:DropdownList do
                      {reveal dl.value
                       case  "文字列" do
                          set params[dl.user-data asa int] = "\'" & {dl.name.remove-clone} &  "\'"
                       else
                          set params[dl.user-data asa int] = {dl.name.remove-clone} asa String
                      }
                  }
              }

    {if {String {val.to-int}} == val then
        {dl.set-value-with-events "数値"}
    elseif  {String {val.to-double}} == val then
        {dl.set-value-with-events "数値"}
    else
        {dl.set-value-with-events "文字列"}
2014-07-03 16:24:18.554999   START:会計年月日　値確定
2014-07-03 16:24:18.563000  DATE  20140413
2014-07-03 16:24:18.570000  PAY-DV  1
[info] #2014/07/03 16:24:18.579# - [SERVICE] [START] DisPayPlnDtService.calc-pay-pln-dt
[info] #2014/07/03 16:24:18.582# - [SERVICE] INPUT-DTO = DisInCalcPayPlnDtDto
[info] #2014/07/03 16:24:18.582# - [SERVICE] PROGRAM-ID = SCRD0000
2014-07-03 16:24:18.746000   START:支払区分　値確定
2014-07-03 16:24:18.848000  DATE  20140703
2014-07-03 16:24:18.848000  PAY-DV  4
[info] #2014/07/03 16:24:18.850# - [SERVICE] [START] DisPayPlnDtService.calc-pay-pln-dt
[info] #2014/07/03 16:24:18.850# - [SERVICE] INPUT-DTO = DisInCalcPayPlnDtDto
[info] #2014/07/03 16:24:18.850# - [SERVICE] PROGRAM-ID = SCRD0001
[info] #2014/07/03 16:24:18.932# - [SERVICE] [END] DisPayPlnDtService.calc-pay-pln-dt
2014-07-03 16:24:18.933999   START:支払区分　値確定
[info] #2014/07/03 16:24:19.055# - [SERVICE] [START] DisPayPlnDtService.calc-pay-pln-dt
2014-07-03 16:24:19.065000   START:会計年月日　値確定

    }

    {return dl}
}

{def type-tbl =
    {Table
        ||height = {make-elastic preferred-size = 1pt},
        ||valign = "top",
        columns = 4}
}


{def formed-params-by-type:StringArray = {StringArray}}

{def parse-cb =
    {CommandButton
        label = "解析",
        {on Action do
            {type-tbl.clear}
            {formed-params-by-type.clear}

            {type-tbl.add "No."}
            {type-tbl.add "Column"}
            {type-tbl.add "Value"}
            {type-tbl.add "Type"}

            def split-sql = {sql-ta.value.merge merge-chars = '?'}

            def params = {param-ta.value.merge mergr-chars = ','}

            {for s in params do
                {formed-params-by-type.append {s.remove-clone}}
            }


            {for s key index:int in params do

                let col:String = ""

                def tmp-ary = {merge-sql[index].merge}

                ||カラム名を探すため、まず。後ろから = を探す
                def find-equal-index =
                    {tmp-ary.find "=", search-direction = SearchDirection.forward}

                {if find-equal-index > 1 then

                    def find-column-index =
                        {tmp-ary.find
                            "",
                            ||空白では「ない」ものを検索
                            equality-proc = {proc {s1:String, s2:String}:bool
                                                {if s1 == s2 then
                                                    {return false}
                                                 And
                                                    {return true}
                                                }
                                            },
                            starting-index = find-equal-index - 1,
                            search-direction = SearchDirection.backward}

                    {if find-column-index> -1 then
                        set col = tmp-ary[find-column-index]
                    }

                }

                {type-tbl.add index + 1}
                {type-tbl.add col}
                {type-tbl.add s}
                {type-tbl.add {get-type-list index, s, formed-params-by-type }}

            }

        }
    }
}

{def replace-cb =
    {CommandButton
        label= "置換",
        {on Action do

            {if not sql-ta.value.empty? then
                def split-sql = {sql-ta.value.split split-chars = '?'}

                let comp-sql:String = ""

                {for s key index:int in formed-params-by-type do
                    set comp-sql = comp-sql & split-sql[index]  & s
                }

                set comp-sql = comp-sql & merge-sql[merge-sql.size - 1]

                set comp-ta.value = comp-sql
            }
        }
    }
