<template>
  <div ref="tuiEditor"></div>
</template>
<script>
import Editor from '@toast-ui/editor';
import editorEvents from './editorEvents';
import valueUpdateMethod from './valueUpdateMethod';

export default {
  name: 'TuiEditor',
  props: {
    previewStyle: {
      type: String
    },
    height: {
      type: String
    },
    value: {
      type: String
    },
    mode: {
      type: String
    },
    options: {
      type: Object
    },
    html: {
      type: String
    },
    visible: {
      type: Boolean,
      default: true
    }
  },
  data() {
    return {
      editor: null
    };
  },
  computed: {
    editorOptions() {
      const options = Object.assign({}, this.options);
      options.initialValue = this.value;
      options.initialEditType = this.mode;
      options.height = this.height;
      options.previewStyle = this.previewStyle;

      return options;
    }
  },
  watch: {
    previewStyle(newValue) {
      this.editor.changePreviewStyle(newValue);
    },
    value(newValue, preValue) {
      if (newValue !== preValue && newValue !== this.editor.getCurrentModeEditor().getValue()) {
        this.editor.getCurrentModeEditor().setValue(newValue);
      }
    },
    height(newValue) {
      this.editor.height(newValue);
    },
    mode(newValue) {
      this.editor.changeMode(newValue);
    },
    html(newValue) {
      this.editor.setHtml(newValue);
      this.$emit('input', this.editor.getCurrentModeEditor().getValue());
    },
    visible(newValue) {
      if (newValue) {
        this.editor.show();
      } else {
        this.editor.hide();
      }
    }
  },
  mounted() {
    const eventOption = {};
    editorEvents.forEach(event => {
      eventOption[event] = (...args) => {
        this.$emit(event, ...args);
      };
    });

    const options = Object.assign(this.editorOptions, {
      el: this.$refs.tuiEditor,
      events: eventOption
    });

    this.editor = new Editor(options);
    if (this.$listeners.input) {
      this.editor.on('change', () => {
        this.$emit('input', this.editor.getCurrentModeEditor().getValue());
      });
    }
  },
  destroyed() {
    editorEvents.forEach(event => {
      this.editor.off(event);
    });
    this.editor.remove();
  },
  methods: {
    invoke(methodName, ...args) {
      let result = null;
      if (this.editor[methodName]) {
        result = this.editor[methodName](...args);
        if (valueUpdateMethod.indexOf(methodName) > -1) {
          this.$emit('input', this.editor.getCurrentModeEditor().getValue());
        }
      }

      return result;
    }
  }
};
</script>
