# クリーンアーキテクチャー　コードを事例に

## 概要

[こちら](https://zenn.dev/tis1116/articles/6c5416e5d77dbf)を参考にクリーンアーキテクチャを紐解く

## `domains`

参考のコードは以下の通り。(言語は TS)

```ts
import moment from "moment-timezone";
import { ID } from "../type";

export class Todo {
  private _id: ID;
  private _title: string;
  private _description: string;
  private _createdAt: moment.Moment;
  private _updatedAt: moment.Moment;

  get id(): ID {
    return this._id;
  }

  set id(id: ID) {
    this._id = id;
  }

  get title(): string {
    return this._title;
  }

  set title(title: string) {
    this._title = title;
  }

  get description(): string {
    return this._description;
  }

  set description(description: string) {
    this._description = description;
  }

  // ビジネスルールを格納
  isTitleFilled(): boolean {
    return this._title.length > 0;
  }

  isDescriptionFilled(): boolean {
    return this._description.length > 0;
  }

  // 省略 日付周りの定義も実施しています

  // idや時刻を生成しない
  // ドメインとしては「タスク」を生成するだけなので、システムで必要なidなどはentityで実施しない
  constructor(title: string, description: string) {
    this._title = title;
    this._description = description;
  }
}
```

- `domeins`はビズネスルールのカプセル化した層
- ここでは`todo`の`id`や`title`などを定義して、値を得たり代入したりルールの格納といった処理をクラスの中に定義している

## `applications`

ここでは CRUD(Create,Read,Update,Delete の頭文字)を定義している。以下例は Create のみ。

```ts
import uuid4 from "uuid4";
import moment from "moment-timezone";
import { Todo } from "../../../domain/Todo";
import { ITodoRepository } from "./ITodoRespository";

export class CreateTodo {
  private taskRepository: ITodoRepository;

  constructor(taskRepository: ITodoRepository) {
    this.taskRepository = taskRepository;
  }

  execute(title: string, description: string) {
    const task = new Todo(title, description);

    if (!task.isTitleFilled() || !task.isDescriptionFilled()) {
      throw new Error("ビジネスルールを破っているためエラー");
    }

    // アプリケーション要件的な要素はインスタンス化で設定せず、setterで設定
    task.id = uuid4();
    task.createdAt = moment();
    task.updatedAt = moment();

    return this.taskRepository.create(task);
  }
}
```

- ここでは`domains`を利用してタスクを生成している
- ここで全体`id`や`createdAt`などをここで生成
- 生成するのは`domains`で可能
- 保存処理はどのように実現するのか？・・・`getways`が担当
