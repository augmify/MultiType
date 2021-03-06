# MultiType
An Android library to retrofit multiple item view types

[![License](https://img.shields.io/badge/license-Apache%202.0-blue.svg)](https://github.com/drakeet/MultiType/blob/master/LICENSE)
![maven-central](https://img.shields.io/maven-central/v/me.drakeet.multitype/multitype.svg)

English Version | <a href="https://drakeet.me/multitype" target="_blank" rel="nofollow">中文版</a>

Previously, when we need to develop a complex RecyclerView/ListView, it is a boring and troublesome work. 
We should override the `getItemViewType` of `RecyclerView.Adapter` and add some types, 
then we create some `ViewHolder` to relate the type, all of the process it is a very bad experience.
**And once we need to add a new type, we have to go to the original Adapter and modify some old codes**, so sad. 

Today, I create a new graceful way to easily develop the complex RecyclerView/ListView, with my MultiType library, 
no matter how complex and how frequently changing list, we could insert a new type without changing the old codes.

## Getting started

In your `build.gradle`:

```groovy
dependencies {
    compile 'me.drakeet.multitype:multitype:1.1'
}
```

## Usage

#### Step 1. Create a class __implements__ `ItemContent`, It would be your `data model`/`Java bean`, for example:

```java
public class TextItemContent implements ItemContent, Savable {

    @NonNull public String text;

    public TextItemContent(@NonNull String text) {
        this.text = text;
    }

    public TextItemContent(@NonNull byte[] data) {
        init(data);
    }

    @Override public void init(@NonNull byte[] data) {
        String json = new String(data);
        this.text = new Gson().fromJson(json, TextItemContent.class).text;
    }

    @NonNull @Override public byte[] toBytes() {
        return new Gson().toJson(this).getBytes();
    }
}
```

#### Step 2. Create a class extends `ItemViewProvider<C extends ItemContent, V extends ViewHolder>`, for example: 

```java
public class TextItemViewProvider
    extends ItemViewProvider<TextItemContent, TextItemViewProvider.TextHolder> {

    static class TextHolder extends RecyclerView.ViewHolder {
        @NonNull final TextView text;

        TextHolder(@NonNull View itemView) {
            super(itemView);
            this.text = (TextView) itemView.findViewById(R.id.text);
        }
    }


    @NonNull @Override
    protected TextHolder onCreateViewHolder(
        @NonNull LayoutInflater inflater, @NonNull ViewGroup parent) {
        View root = inflater.inflate(R.layout.item_text, parent, false);
        TextHolder holder = new TextHolder(root);
        return holder;
    }


    @Override
    protected void onBindViewHolder(
        @NonNull TextHolder holder, @NonNull TextItemContent content, @NonNull TypeItem typeItem) {
        holder.text.setText("hello: " + content.text);
    }
}
```

#### Step 3. You do not need to create another new class. Just add a `RecyclerView` and `List<TypeItem>` to your `Activity`, for example: 

```java
@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);
    recyclerView = (RecyclerView) findViewById(R.id.list);
    
    itemFactory = new TypeItemFactory.Builder().build();
    TypeItem textItem = itemFactory.newItem(new TextItemContent("world"));
    TypeItem imageItem = itemFactory.newItem(new ImageItemContent(R.mipmap.ic_launcher));
    TypeItem richItem = itemFactory.newItem(new RichItemContent("小艾大人赛高", R.mipmap.avatar));
    
    List<TypeItem> typeItems = new ArrayList<>(80);
    for (int i = 0; i < 20; i++) {
        typeItems.add(textItem);
        typeItems.add(imageItem);
        typeItems.add(richItem);
    }

    /* register the types before setAdapter, that's all right */ 
    ItemTypePool.register(TextItemContent.class, new TextItemViewProvider());
    ItemTypePool.register(ImageItemContent.class, new ImageItemViewProvider());
    ItemTypePool.register(RichItemContent.class, new RichItemViewProvider());

    recyclerView.setAdapter(new MultiTypeAdapter(typeItems));
}
```

**You're good to go!** 

You could check the `sample` module for more details and after running it will look like: 

<img src="art/screenshot.png" width=270 height=486/>

License
-------

    Copyright 2016 drakeet.

    Licensed under the Apache License, Version 2.0 (the "License");
    you may not use this file except in compliance with the License.
    You may obtain a copy of the License at

       http://www.apache.org/licenses/LICENSE-2.0

    Unless required by applicable law or agreed to in writing, software
    distributed under the License is distributed on an "AS IS" BASIS,
    WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
    See the License for the specific language governing permissions and
    limitations under the License.





