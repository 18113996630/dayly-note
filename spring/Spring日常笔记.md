## Spring日常笔记

1. 在**controller**中接收**Date**类型的参数

   ```
   @InitBinder
   protected void init(WebDataBinder binder) {
   	SimpleDateFormat dateFormat = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
   	dateFormat.setLenient(false);
   	binder.registerCustomEditor(Date.class, new CustomDateEditor(dateFormat, false));
   }
   ```

   