<template>
  <div class="container">
    <div class="hello">
      <h1>{{ msg }}</h1>
    </div>
    <div class="col-md-6">
      <b-form-textarea id="textarea1"
                       v-model="xmlStr"
                       placeholder="Enter XML Generic from Texture Packer"
                       :rows="20"
                       :max-rows="20"
                       no-resize={true}
      >
      </b-form-textarea>
      <b-form-input v-model="fontFace"
                    type="text"
                    placeholder="Enter font face (font name)">
      </b-form-input>
      <b-button class="convert-button"
                v-on:click="convert"
      >
        Convert
      </b-button>
    </div>
    <div class="col-md-6">
      <b-form-textarea id="textarea1"
                       v-model="outputXml"
                       placeholder="Output"
                       :rows="20"
                       :max-rows="20"
                       no-resize={true}
      >
      </b-form-textarea>
    </div>
  </div>
</template>

<script>
  import * as xml from '../utils/xml';
  import * as StringUtils from '../utils/str';
  const BASE_FNT = `<font>
            <info face="{faceName}" size="63" bold="0" italic="0" charset="" unicode="" stretchH="100" smooth="1" aa="1" padding="0,0,0,0" spacing="0,0" outline="0"/>
            <common lineHeight="{height}" base="{base}" scaleW="{width}" scaleH="{height}" pages="1" packed="0"/>
            <pages>
              <page id="0" file="{fileName}"/>
            </pages>
            <chars count="{counter}">
            {listChar}
            </chars>
            <kernings count="1">
              <kerning first="49" second="49" amount="-2"/>
            </kernings>
          </font>`;
  export default {
    name: 'HelloWorld',
    data() {
      return {
        msg: 'Convert Texture Packer Generic XML to FNT (XML)',
        xmlStr: '',
        outputXml: '',
        fontFace: '',
      };
    },
    methods: {
      convert() {
        let parser = new DOMParser();
        let xmlDoc = parser.parseFromString(this.xmlStr, 'text/xml');
        let textureAtlas = xmlDoc.querySelector('TextureAtlas');
        let imagePath = xml.attr(textureAtlas, 'imagePath');
        let width = xml.attr(textureAtlas, 'width');
        let height = xml.attr(textureAtlas, 'height');
        let sprites = textureAtlas.querySelectorAll('sprite');
        let spritesData = [];
        let output = BASE_FNT;
        let listChar = ``;
        sprites.forEach((sprite)=>{
          let spriteData = {
            name: xml.attr(sprite, 'n'),
            x: parseInt(xml.attr(sprite, 'x')),
            y: parseInt(xml.attr(sprite, 'y')),
            w: parseInt(xml.attr(sprite, 'w')),
            h: parseInt(xml.attr(sprite, 'h')),
          };
          spritesData.push(spriteData);
          let yOffset = height-spriteData.h;
          let xAdvance = spriteData.w + +2;

          listChar += '\n<char id="'+ spriteData.name.split('.')[0] +'" x="' + spriteData.x + '" y="' + spriteData.y + '" width="' + spriteData.w + '" height="' + spriteData.h + '" ' +
            'xoffset="1" yoffset="' + yOffset + '" xadvance="' + xAdvance + '" page="0" chnl="15"/>';
        });

        output = StringUtils.replaceAll(output, '{faceName}', this.fontFace);
        output = StringUtils.replaceAll(output, '{height}', height);
        output = StringUtils.replaceAll(output, '{width}', width);
        output = StringUtils.replaceAll(output, '{base}', 30);
        output = StringUtils.replaceAll(output, '{fileName}', imagePath);
        output = StringUtils.replaceAll(output, '{counter}', sprites.length);
        output = StringUtils.replaceAll(output, '{listChar}', listChar);
        // let outXml = parser.parseFromString(output, 'text/xml');

        // outXml.getElementsByTagName('font')[0].appendChild(info);

        this.outputXml = output;
      },
    },
  };

</script>

<style scoped>
  h1, h2 {
    font-weight: normal;
  }

  ul {
    list-style-type: none;
    padding: 0;
  }

  li {
    display: inline-block;
    margin: 0 10px;
  }

  .convert-button {
    margin-top: 20px;
  }

  a {
    color: #42b983;
  }
</style>
