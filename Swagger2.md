# Swagger2导出接口文档

```java
   /**
     * 生成Markdown格式文档
     * @throws Exception
     */
    @Test
    public void generateMarkdownDocs() throws Exception {
        //    输出Markdown格式
        Swagger2MarkupConfig config = new Swagger2MarkupConfigBuilder()
                .withMarkupLanguage(MarkupLanguage.MARKDOWN)
                .withOutputLanguage(Language.ZH)
                .withPathsGroupedBy(GroupBy.TAGS)
                .withGeneratedExamples()
                .withoutInlineSchema()
                .build();

        Swagger2MarkupConverter.from(new URL("http://localhost:8580/cwtqss/v2/api-docs?group=System"))
                .withConfig(config)
                .build()
                .toFolder(Paths.get("./docs/markdown/generated"));
    }

   /**
     * 生成Markdown格式文档,并汇总成一个文件
     * @throws Exception
     */
    @Test
    public void generateMarkdownDocsToFile() throws Exception {
        //    输出Markdown到单文件
        Swagger2MarkupConfig config = new Swagger2MarkupConfigBuilder()
                .withMarkupLanguage(MarkupLanguage.MARKDOWN)
                .withOutputLanguage(Language.ZH)
                .withPathsGroupedBy(GroupBy.TAGS)
                .withGeneratedExamples()
                .withoutInlineSchema()
                .build();
 
        Swagger2MarkupConverter.from(new URL("http://localhost:8580/cwtqss/v2/api-docs?group=System"))
                .withConfig(config)
                .build()
                .toFile(Paths.get("./docs/markdown/generated/all"));
    }


    /**
     * 生成AsciiDocs格式文档
     * @throws Exception
     */
    @Test
    public void generateAsciiDocs() throws Exception {
        //    输出Ascii格式
        Swagger2MarkupConfig config = new Swagger2MarkupConfigBuilder()
                .withMarkupLanguage(MarkupLanguage.ASCIIDOC)
                .withOutputLanguage(Language.ZH)
                .withPathsGroupedBy(GroupBy.TAGS)
                .withGeneratedExamples()
                .withoutInlineSchema()
                .build();

        Swagger2MarkupConverter.from(new URL("http://localhost:8580/cwtqss/v2/api-docs?group=System"))
                .withConfig(config)
                .build()
                .toFolder(Paths.get("./docs/asciidoc/generated"));
    }
    /**
      * 生成AsciiDocs格式文档,并汇总成一个文件
      * @throws Exception
      */
    @Test
    public void generateAsciiDocsToFile() throws Exception {
        //    输出Ascii到单文件
        Swagger2MarkupConfig config = new Swagger2MarkupConfigBuilder()
            .withMarkupLanguage(MarkupLanguage.ASCIIDOC)
            .withOutputLanguage(Language.ZH)
            .withPathsGroupedBy(GroupBy.TAGS)
            .withGeneratedExamples()
            .withoutInlineSchema()
            .build();

        Swagger2MarkupConverter.from(new URL("http://localhost:8580/cwtqss/v2/api-docs?group=System"))
            .withConfig(config)
            .build()
            .toFile(Paths.get("./docs/asciidoc/generated/all"));
    }





```

