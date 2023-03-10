```tsx
import React, { useState } from 'react';
import classNames from 'classnames';
import Input, { IInputProps } from '../input';

interface TreeNode {
  value: string;
  label: string;
  children?: TreeNode[];
}

export interface TreeSelectProps extends Omit<IInputProps, 'onChange'>{
  options: TreeNode[];
  prefixCls?: string;
  value?: string;
  placeholder?: string;
  onChange?: (value: string) => void;
  disabled?: boolean;
}

const TreeSelect: React.FC<TreeSelectProps> = ({
  prefixCls,
  options,
  value,
  onChange,
  disabled,
  ...rest
}) => {
  const [isOpen, setIsOpen] = useState(false);
  const [selectedValue, setSelectedValue] = useState(value);

  const handleInputChange = () => {
    setIsOpen((prevIsOpen) => !prevIsOpen);
  };

  const handleNodeClick = (e, value: string, disabled: boolean) => {
    // 阻止冒泡必须要在最前面，不然就算disabled，也依然会触发父级的click事件
    e.stopPropagation();
    if (disabled) return;
    setSelectedValue(value);
    setIsOpen(false);
    onChange && onChange(value);
  };

  const renderTreeNodes = (nodes: TreeNode[]) => {
    return nodes.map((node) => {
      const hasChildren = node.children && node.children.length > 0;
      const isSelected = selectedValue === node.value;
      const isDisabled = disabled;

      const nodeClasses = classNames('tree-select__tree-item', {
        'tree-select__tree-item--no-children': !hasChildren,
        'tree-select__tree-item--selected': isSelected,
        'tree-select__tree-item--disabled': isDisabled,
      });

      return (
        <li key={node.value} className={nodeClasses} onClick={(e) => handleNodeClick(e, node.value, disabled)}>
          <div className="tree-select__tree-item-label">{node.label}</div>
          {hasChildren && (
            <div
              className={classNames('tree-select__tree-item-icon', {
                'tree-select__tree-item-icon--open': isOpen,
              })}
            />
          )}
          {hasChildren && isOpen && (
            <ul className="tree-select__sub-tree">{renderTreeNodes(node.children!)}</ul>
          )}
        </li>
      );
    });
  };

  const treeSelectClasses = classNames(prefixCls, {
    [`${prefixCls}--open`]: isOpen,
    [`${prefixCls}--disabled`]: disabled,
  });

  return (
    <div className={treeSelectClasses}>
      <div className={`${prefixCls}__input`} onClick={!disabled && handleInputChange}>
        {/* <input type="text" readOnly value={selectedValue || ''} placeholder={placeholder} /> */}
        <Input
          value={selectedValue || ''}
          readOnly
          {...rest}
        />
        <div
          className={classNames(`${prefixCls}__input-icon`, {
            [`${prefixCls}__input-icon--open`]: isOpen,
          })}
        />
      </div>
      {isOpen && (
        <ul className={`${prefixCls}__tree`}>
          {renderTreeNodes(options)}
        </ul>
      )}
    </div>
  );
};

TreeSelect.defaultProps = {
  prefixCls: 'oned-tree-select',
  disabled: false
};

export default TreeSelect;
```