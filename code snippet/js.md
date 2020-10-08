# 1. 将 table 内的 input 数据转换为 json

```js
function tableConverJson(tableId) {
  var jsonStr = "{";
  $('#'+ tableId +' tbody').find('tr').each(function () { //tableId 是table表格的id
    $(this).find('td').each(function () {
      $(this).find('input').each(function () {  //获取td中input的值
        if($(this).attr("id")) { //json 的 key的值
          jsonStr += "\""+$(this).attr("id")+"\":\""+$(this).val()+"\",";
        }
      })
    })
    jsonStr = jsonStr.substring(0,jsonStr.length - 1);
  })
  jsonStr = jsonStr.substring(0,jsonStr.length - 1);
  jsonStr += "}";
  return jsonStr;
}
```

