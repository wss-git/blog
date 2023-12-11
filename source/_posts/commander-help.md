---
title: commander 自定义 Help
categories:
  - 轮子应用
excerpt: 'commander 自定义 Help'
description: 'commander 自定义 Help'
date: 2023-12-11 10:10:57
---


```js
const { Command } = require('commander');
const program = new Command();

program.name('test').description('CLI to some JavaScript string utilities');

// 配置系统命令： split
const split = program.command(
  'split',
  { hidden: true }, // help 时隐藏说明
);
split.helpInformation = function () {
  return '';
};
split
  .description('Split a string into substrings and display as an array')
  .summary(`Add an account`) // 作为子命令时简略输出
  .argument('<string>', 'string to split')
  .option('--first', 'display just the first substring')
  .action((str, options) => {
    const limit = options.first ? 1 : undefined;
    console.log(str.split(options.separator, limit));
  })
  .on('--help', () => {
    console.log('子命令 split');
  });

// 配置系统命令：config
const config = program.command('config');
config.helpInformation = function () {
  return '';
};
config
  .description('Split a string into substrings and display as an array')
  .argument('<string>', 'string to split')
  .action((str, options) => {
    const limit = options.first ? 1 : undefined;
    console.log(str.split(options.separator, limit));
  })
  .on('--help', () => {
    console.log('子命令 config');
  });

// 配置自定义命令
const customCommand = process.argv[2];
if (customCommand && !customCommand.startsWith('-')) {
  // 动态命令配置
  const subProgram = program.command(customCommand);
  subProgram.helpInformation = function () {
    console.log(`自定义子命令`);
    return '';
  };
  subProgram
    .description('custom Args')
    .argument('<string>', 'string to split')
    .option('--first', 'display just the first substring')
    .action((str, options) => {
      console.log('subProgram: ', str, options);
    });
  subProgram.allowUnknownOption();
}

program.helpInformation = function () {
  return '';
};

program.on('--help', function () {
  console.log('根命令');
});

program.parse();
```

