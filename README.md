# JPMC Task 3
Starter repo for task 3 of JPMC's Forage program
changes made in graph.tsx are

import React, { Component } from 'react';
import { Table } from '@finos/perspective';
import { ServerRespond } from './DataStreamer';
import { DataManipulator } from './DataManipulator';
import './Graph.css';

interface IProps {
  data: ServerRespond[],
}

interface PerspectiveViewerElement extends HTMLElement {
  load: (table: Table) => void,
}
class Graph extends Component<IProps, {}> {
  table: Table | undefined;

  render() {
    return React.createElement('perspective-viewer');
  }

  componentDidMount() {
    // Get element from the DOM.
    const elem = document.getElementsByTagName('perspective-viewer')[0] as unknown as PerspectiveViewerElement;

    const schema = {
     price_abc: 'float',
    price_def: 'float',
    ratio: 'float',
    timestamp: 'date',
    upper_bound: 'float',
    lower_bound: 'float',
    trigger_alert: 'float', 
 };

    if (window.perspective && window.perspective.worker()) {
      this.table = window.perspective.worker().table(schema);
    }
    if (this.table) {
      // Load the `table` in the `<perspective-viewer>` DOM reference.
      elem.load(this.table);
      elem.setAttribute('view', 'y_line');
      elem.setAttribute('column-pivots', '["stock"]');
      elem.setAttribute('row-pivots', '["timestamp"]');
      elem.setAttribute('columns', '["top_ask_price"]');
      elem.setAttribute('aggregates', JSON.stringfy({
      // To configure our graph, modify/add more attributes to the element. 
      price_abc: 'avg',
      price_def: 'avg',
      ratio: 'avg',
      timestamp: 'distinct count',
      upper_bound: 'avg',
      lower_bound: 'avg',
      trigger_alert: 'avg',
      }));
    }
  }

  componentDidUpdate() {
    if (this.table) {
      this.table.update([
        DataManipulator.generateRow(this.props.data),
      ]);
    }
  }
}

export default Graph;

changes made in datamanipulators.ts are

import { ServerRespond } from './DataStreamer';

export interface Row {
 price_abc: number,
  price_def: number,
  ratio: number,
  timestamp: Date,
  upper_bound: number,
  lower_bound: number,
  trigger_alert: number | undefined, 
}


export class DataManipulator {
 static generateRow(serverResponds: ServerRespond[]): Row {
    const priceABC = (serverResponds[0].top_ask.price + serverResponds[0].top_bid.price) / 2;
    const priceDEF = (serverResponds[1].top_ask.price + serverResponds[1].top_bid.price) / 2;
    const ratio = priceABC / priceDEF;
    const uperBound = 1 + 0.05;
    const lowerBound = 1 - 0.05
    return {
      price_abc: priceABC,
      price_def: priceDEF,
      ratio,
      timestamp: serverResponds[0].timestamp > serverResponds[1].timestamp ?
          serverResponds[0].timestamp : serverResponds[1].timestamp,
      upper_bound: uperBound,
      lower_bound: lowerBound,
      trigger_alert: (ratio > uperBound || ratio < lowerBound) ? ratio : undefined, 
      };
    }
  }

