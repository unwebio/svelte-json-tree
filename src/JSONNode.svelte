<script>
  import JSONObjectNode from './JSONObjectNode.svelte';
  import JSONArrayNode from './JSONArrayNode.svelte';
  import JSONIterableArrayNode from './JSONIterableArrayNode.svelte';
  import JSONIterableMapNode from './JSONIterableMapNode.svelte';
  import JSONMapEntryNode from './JSONMapEntryNode.svelte';
  import JSONValueNode from './JSONValueNode.svelte';
  import ErrorNode from './ErrorNode.svelte';
  import objType from './objType';

  export let key, value, isParentExpanded, isParentArray;
  $: nodeType = objType(value);
  $: componentType = getComponent(nodeType);
  $: valueGetter = getValueGetter(nodeType);

  function getComponent(nodeType) {
    switch (nodeType) {
      case 'Object':
        return JSONObjectNode;
      case 'Error':
        return ErrorNode;
      case 'Array':
        return JSONArrayNode;
      case 'Iterable':
      case 'Map':
      case 'Set':
        return typeof value.set === 'function' ? JSONIterableMapNode : JSONIterableArrayNode;
      case 'MapEntry':
        return JSONMapEntryNode;
      default:
        if (nodeType !== 'Function' && value instanceof Object) {
          return JSONObjectNode;
        }
        return JSONValueNode;
    }
  }

  function getValueGetter(nodeType) {
    switch (nodeType) {
      case 'Object':
      case 'Error':
      case 'Array':
      case 'Iterable':
      case 'Map':
      case 'Set':
      case 'MapEntry':
      case 'Number':
        return undefined;
      case 'String':
        return raw => `"${raw}"`;
      case 'Boolean':
        return raw => (raw ? 'true' : 'false');
      case 'Date':
        return raw => raw.toISOString();
      case 'Null':
        return () => 'null';
      case 'Undefined':
        return () => 'undefined';
      case 'Function':
      case 'Symbol':
        return raw => raw.toString();
      default:
        if (value instanceof Object) {
          return undefined;
        }
        return () => `<${nodeType}>`;
    }
  }
</script>

<svelte:component this={componentType} {key} {value} {isParentExpanded} {isParentArray} {nodeType} {valueGetter} />
