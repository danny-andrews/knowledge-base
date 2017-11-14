TODO:
- Determine PROs and CONs of HoCs and component enhancers
- Determine capabilities of HoCs and component enhancers
- Based on which one is superior, favor it except in cases where it does not suffice

HoC capabilities:
- Manipulate props
- Manipulate state
- Abstract state
- Access the instance via Refs
- Wrap the WrappedComponent with other elements
- Highjack render (isEditable use-case)
  - Read, add, edit, remove props in any of the React Elements outputted by render
  - Read, and modify the React Elements tree outputted by render
  - Conditionally display the elements tree
  - Wrap the element’s tree for styling purposes (as shown in Props Proxy)

References:
- https://medium.com/@franleplant/react-higher-order-components-in-depth-cf9032ee6c3e#5247
