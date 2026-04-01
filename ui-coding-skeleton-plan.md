# ICAR-4389 UI Coding Skeleton（组件拆分代码骨架清单）

## 1. 目标

- 给前端开发一份可直接开工的代码骨架清单。
- 对齐现有 OWS 技术栈与组件复用策略。
- 明确每个文件的职责、Props、状态与接口映射。

---

## 2. 推荐目录结构

```text
src/pages/partnerManagement/commissionExchangeRate/
├── index.tsx
├── style.module.less
├── constants.ts
├── types.ts
├── hooks/
│   └── useCommissionExchangeRate.ts
├── components/
│   ├── BrandTabs.tsx
│   ├── ExchangeRateSettingCard.tsx
│   ├── ExchangeRateRecordTable.tsx
│   └── BindTwoFaDialogAdapter.tsx
└── services/
    └── api.ts
```

---

## 3. 文件级代码骨架

## 3.1 types.ts

```ts
export type BrandCode = 'VG' | 'VT' | 'PU' | 'MM' | 'UM' | 'VTJ' | 'ST';

export interface RateConfig {
  crossCurrencyRate?: number;
  pipPercentRate?: number;
  perMillionRate?: number;
  operator?: string;
  updateTime?: string;
}

export interface RateRecord {
  id: string;
  brand: BrandCode;
  from: RateConfig | null;
  to: RateConfig;
  operator: string;
  time: string;
}

export interface QueryRecordParams {
  brand: BrandCode;
  pageNum?: number;
  pageSize?: number;
}
```

## 3.2 constants.ts

```ts
import type { BrandCode } from './types';

export const BRAND_TABS: BrandCode[] = ['VG', 'VT', 'PU', 'MM', 'UM', 'VTJ', 'ST'];

export const RATE_FIELD_KEYS = {
  crossCurrencyRate: 'crossCurrencyRate',
  pipPercentRate: 'pipPercentRate',
  perMillionRate: 'perMillionRate',
} as const;

export const RATE_RANGE = {
  min: 0.01,
  max: 10,
  precision: 2,
};
```

## 3.3 services/api.ts

```ts
import type { BrandCode, QueryRecordParams, RateConfig, RateRecord } from '../types';

export async function apiGetRateConfig(brand: BrandCode): Promise<RateConfig> {
  throw new Error('todo');
}

export async function apiSaveRateConfig(payload: {
  brand: BrandCode;
  config: RateConfig;
}): Promise<void> {
  throw new Error('todo');
}

export async function apiGetRateRecords(params: QueryRecordParams): Promise<{
  total: number;
  rows: RateRecord[];
}> {
  throw new Error('todo');
}

export async function apiTwoFactorStatus(params: any): Promise<any> {
  throw new Error('todo');
}

export async function apiTwoFactorEnable(params: any): Promise<any> {
  throw new Error('todo');
}

export async function apiTwoFapreValidate(params: any): Promise<any> {
  throw new Error('todo');
}

export async function apichangeDevice(params: any): Promise<any> {
  throw new Error('todo');
}

```

## 3.4 hooks/useCommissionExchangeRate.ts

```ts
import { useCallback, useMemo, useState } from 'react';
import { message } from 'antd';
import {
  apiGetRateConfig,
  apiGetRateRecords,
  apiSaveRateConfig,
} from '../services/api';
import type { BrandCode, RateConfig, RateRecord } from '../types';

export function useCommissionExchangeRate() {
  const [brand, setBrand] = useState<BrandCode>('VG');
  const [loading, setLoading] = useState(false);
  const [saving, setSaving] = useState(false);
  const [config, setConfig] = useState<RateConfig>({});
  const [records, setRecords] = useState<RateRecord[]>([]);
  const [total, setTotal] = useState(0);

  const fetchConfig = useCallback(async (nextBrand: BrandCode) => {
    setLoading(true);
    try {
      const data = await apiGetRateConfig(nextBrand);
      setConfig(data);
    }
    finally {
      setLoading(false);
    }
  }, []);

  const fetchRecords = useCallback(async (nextBrand: BrandCode) => {
    setLoading(true);
    try {
      const res = await apiGetRateRecords({ brand: nextBrand, pageNum: 1, pageSize: 20 });
      setRecords(res.rows);
      setTotal(res.total);
    }
    finally {
      setLoading(false);
    }
  }, []);

  const changeBrand = useCallback(async (nextBrand: BrandCode) => {
    setBrand(nextBrand);
    await Promise.all([fetchConfig(nextBrand), fetchRecords(nextBrand)]);
  }, [fetchConfig, fetchRecords]);

  const saveConfig = useCallback(async (nextConfig: RateConfig) => {
    setSaving(true);
    try {
      await apiSaveRateConfig({ brand, config: nextConfig });
      message.success('Saved successfully');
      await Promise.all([fetchConfig(brand), fetchRecords(brand)]);
    }
    finally {
      setSaving(false);
    }
  }, [brand, fetchConfig, fetchRecords]);

  return useMemo(() => ({
    brand,
    loading,
    saving,
    config,
    records,
    total,
    setConfig,
    changeBrand,
    fetchConfig,
    fetchRecords,
    saveConfig,
  }), [brand, loading, saving, config, records, total, changeBrand, fetchConfig, fetchRecords, saveConfig]);
}
```

## 3.5 components/BrandTabs.tsx

```tsx
import { Tabs } from 'antd';
import { BRAND_TABS } from '../constants';
import type { BrandCode } from '../types';

interface BrandTabsProps {
  activeBrand: BrandCode;
  onChange: (brand: BrandCode) => void;
}

const BrandTabs = ({ activeBrand, onChange }: BrandTabsProps) => {
  return (
    <Tabs
      activeKey={activeBrand}
      onChange={(key) => onChange(key as BrandCode)}
      items={BRAND_TABS.map(item => ({ key: item, label: item }))}
      type="line"
      size="large"
    />
  );
};

export default BrandTabs;
```

## 3.6 components/ExchangeRateSettingCard.tsx

```tsx
import { Button, Form, InputNumber, Tooltip } from 'antd';
import { QuestionCircleOutlined } from '@ant-design/icons';
import type { RateConfig } from '../types';

interface ExchangeRateSettingCardProps {
  brandLabel: string;
  value: RateConfig;
  editing: boolean;
  saving: boolean;
  onEdit: () => void;
  onCancel: () => void;
  onSave: (value: RateConfig) => void;
}

const ExchangeRateSettingCard = ({
  brandLabel,
  value,
  editing,
  saving,
  onEdit,
  onCancel,
  onSave,
}: ExchangeRateSettingCardProps) => {
  const [form] = Form.useForm<RateConfig>();

  return (
    <div>
      <h3>{brandLabel} Exchange Rate Setting</h3>
      <Form form={form} initialValues={value} onFinish={onSave} layout="vertical">
        <Form.Item name="crossCurrencyRate" label="Cross Currency Rate">
          <InputNumber min={0.01} max={10} precision={2} suffix="%" />
        </Form.Item>
        <Form.Item name="pipPercentRate" label="Pip_% Rate">
          <InputNumber min={0.01} max={10} precision={2} suffix="%" />
        </Form.Item>
        <Form.Item name="perMillionRate" label="Per Million Rate">
          <InputNumber min={0.01} max={10} precision={2} suffix="%" />
        </Form.Item>
        <Tooltip title="rate tips">
          <QuestionCircleOutlined />
        </Tooltip>
        {editing
          ? (
              <>
                <Button onClick={onCancel}>Cancel</Button>
                <Button type="primary" htmlType="submit" loading={saving}>Save</Button>
              </>
            )
          : <Button onClick={onEdit}>Edit</Button>}
      </Form>
    </div>
  );
};

export default ExchangeRateSettingCard;
```

## 3.7 components/ExchangeRateRecordTable.tsx

```tsx
import { BusinessTable } from '@hytechc/admin-ui';
import type { BusinessTableProps } from '@hytechc/admin-ui';
import type { RateRecord } from '../types';

interface ExchangeRateRecordTableProps {
  loading: boolean;
  data: RateRecord[];
  total: number;
}

const ExchangeRateRecordTable = ({ loading, data }: ExchangeRateRecordTableProps) => {
  const columns: BusinessTableProps<RateRecord>['columns'] = [
    { title: 'From', dataIndex: 'from' },
    { title: 'To', dataIndex: 'to' },
    { title: 'Operator', dataIndex: 'operator' },
    { title: 'Time', dataIndex: 'time' },
  ];

  return (
    <BusinessTable
      title="Exchange Rate Record"
      columns={columns}
      dataSource={data}
      loading={loading}
      rowKey="id"
      search={false}
      pagination={false}
    />
  );
};

export default ExchangeRateRecordTable;
```

## 3.8 components/BindTwoFaDialogAdapter.tsx

```tsx
import { BindTwoFaDialog } from '@hytechc/business';

interface BindTwoFaDialogAdapterProps {
  open: boolean;
  onCancel: () => void;
  onSuccess: () => void;
  onTwoFactorStatus: (params: any) => Promise<any>;
  onTwoFactorEnable: (params: any) => Promise<any>;
  onTwoFapreValidate: (params: any) => Promise<any>;
  onchangeDevice: (params: any) => Promise<any>;
}

const BindTwoFaDialogAdapter = ({
  open,
  onCancel,
  onSuccess,
  onTwoFactorStatus,
  onTwoFactorEnable,
  onTwoFapreValidate,
  onchangeDevice,
}: BindTwoFaDialogAdapterProps) => {
  return (
    <BindTwoFaDialog
      visible={open}
      onClose={onCancel}
      closeCallback={onCancel}
      onTwoFactorStatus={onTwoFactorStatus}
      onTwoFactorEnable={onTwoFactorEnable}
      onTwoFapreValidate={onTwoFapreValidate}
      onchangeDevice={onchangeDevice}
      type="business"
      successCallback={onSuccess}
    />
  );
};

export default BindTwoFaDialogAdapter;
```

## 3.9 index.tsx

```tsx
import { useEffect, useState } from 'react';
import BrandTabs from './components/BrandTabs';
import ExchangeRateSettingCard from './components/ExchangeRateSettingCard';
import ExchangeRateRecordTable from './components/ExchangeRateRecordTable';
import BindTwoFaDialogAdapter from './components/BindTwoFaDialogAdapter';
import { useCommissionExchangeRate } from './hooks/useCommissionExchangeRate';
import {
  apichangeDevice,
  apiTwoFactorEnable,
  apiTwoFapreValidate,
  apiTwoFactorStatus,
} from './services/api';

const CommissionExchangeRatePage = () => {
  const {
    brand,
    loading,
    saving,
    config,
    records,
    total,
    setConfig,
    changeBrand,
    saveConfig,
    fetchConfig,
    fetchRecords,
  } = useCommissionExchangeRate();
  const [editing, setEditing] = useState(false);
  const [open2FA, setOpen2FA] = useState(false);
  const [pendingConfig, setPendingConfig] = useState(config);

  useEffect(() => {
    fetchConfig(brand);
    fetchRecords(brand);
  }, [brand, fetchConfig, fetchRecords]);

  return (
    <div>
      <BrandTabs activeBrand={brand} onChange={changeBrand} />
      <ExchangeRateSettingCard
        brandLabel={brand}
        value={config}
        editing={editing}
        saving={saving}
        onEdit={() => setEditing(true)}
        onCancel={() => {
          setConfig(config);
          setEditing(false);
        }}
        onSave={(nextValue) => {
          setPendingConfig(nextValue);
          setOpen2FA(true);
        }}
      />
      <ExchangeRateRecordTable loading={loading} data={records} total={total} />
      <BindTwoFaDialogAdapter
        open={open2FA}
        onCancel={() => setOpen2FA(false)}
        onTwoFactorStatus={apiTwoFactorStatus}
        onTwoFactorEnable={apiTwoFactorEnable}
        onTwoFapreValidate={apiTwoFapreValidate}
        onchangeDevice={apichangeDevice}
        onSuccess={async () => {
          await saveConfig(pendingConfig);
          setOpen2FA(false);
          setEditing(false);
        }}
      />
    </div>
  );
};

export default CommissionExchangeRatePage;
```

---

## 4. 编码顺序建议

- 第 1 步：先落 `types.ts` + `constants.ts` + `services/api.ts`
- 第 2 步：实现 `useCommissionExchangeRate.ts`，保证数据流先通
- 第 3 步：接入 `BrandTabs`（仅品牌 Tab，不带右侧下拉）
- 第 4 步：实现配置卡片 + 二次验证弹窗
- 第 5 步：实现历史记录表
- 第 6 步：接入权限与国际化 key

---

## 5. 开发完成检查项

- Tab 切换会刷新配置与记录
- Edit / Save / Cancel 状态切换正确
- 百分比输入校验：可空、0.01~10、2位小数
- Save 必须经过 2FA
- 保存成功后历史记录刷新且最新在前
- 无编辑权限时仅可查看

---

## 6. 文件与 REQ 映射（代码审计入口）

| 文件 | 覆盖需求ID | 审查重点 |
|---|---|---|
| `index.tsx` | REQ-004, REQ-005 | 状态流转与 2FA 串联 |
| `components/BrandTabs.tsx` | REQ-003, REQ-004 | 仅品牌 Tab，无右侧下拉 |
| `components/ExchangeRateSettingCard.tsx` | REQ-003, REQ-004 | 三字段校验与编辑态 |
| `components/ExchangeRateRecordTable.tsx` | REQ-006 | 字段展示与排序 |
| `components/BindTwoFaDialogAdapter.tsx` | REQ-005 | 2FA 组件参数透传正确 |
| `hooks/useCommissionExchangeRate.ts` | REQ-003, REQ-004, REQ-006 | 请求编排与刷新策略 |
| `services/api.ts` | REQ-003, REQ-005, REQ-006 | 接口签名与请求参数一致 |

---

## 7. Coding Harness 最小清单

- [ ] 每个变更文件可反查到至少一个 `REQ-*`
- [ ] PR 描述包含 `Implements: REQ-*`
- [ ] PR 描述包含 `Tests: case-*`
- [ ] lint/typecheck/test 全绿后才允许合并
- [ ] 若 UI 交互变更，需同步更新 `ui-analysis-with-components.md`
