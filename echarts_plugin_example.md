# Hướng dẫn tạo Custom Plugin ECharts cho Open MCT

Dưới đây là ví dụ hoàn chỉnh về cách tạo một Custom Plugin trong Open MCT sử dụng thư viện **Apache ECharts** để render dữ liệu thay thế cho hệ thống Plot mặc định. 

Chúng ta sẽ tạo 3 phần chính giống hệt cấu trúc gốc của Open MCT nhưng đơn giản hóa phần View:
1. `EchartsPlugin.js`: Đăng ký plugin với Open MCT.
2. `EchartsViewProvider.js`: Khởi tạo View khi user mở component.
3. `EchartsComponent.vue`: Nơi ECharts chạy và nhận data từ Telemetry API.

---

## 1. File gốc: EchartsPlugin.js
File này định nghĩa loại object mới tên là `echarts-plot` và cho phép user kéo thả telemetry vào (qua composition).

```javascript
import EchartsViewProvider from './EchartsViewProvider.js';

export default function EchartsPlugin() {
    return function install(openmct) {
        // 1. Đăng ký một View Provider cho ECharts
        openmct.objectViews.addProvider(new EchartsViewProvider(openmct));

        // 2. Định nghĩa một type mới trong menu Create
        openmct.types.addType('echarts-plot', {
            name: 'ECharts Plot',
            creatable: true,
            description: 'Biểu đồ sử dụng thư viện ECharts để hiển thị dữ liệu đẹp hơn.',
            cssClass: 'icon-telemetry',
            initialize(domainObject) {
                // Khởi tạo composition rỗng để user có thể kéo Telemetry vào
                domainObject.composition = [];
            }
        });

        // 3. Cho phép kéo thả vào composition
        openmct.composition.addPolicy((parent, child) => {
            if (parent.type === 'echarts-plot') {
                return openmct.telemetry.isTelemetryObject(child);
            }
            return true;
        });
    };
}
```

---

## 2. File: EchartsViewProvider.js
Tương tự như [GaugeViewProvider](file:///d:/work/openmct/openmct/src/plugins/gauge/GaugeViewProvider.js#27-75), file này định nghĩa điều kiện hiển thị component Vue.

```javascript
import { createApp } from 'vue'; // Dùng Vue 3 theo chuẩn Open MCT mới
import EchartsComponent from './EchartsComponent.vue';

export default function EchartsViewProvider(openmct) {
    return {
        key: 'echarts-plot-view',
        name: 'ECharts View',
        cssClass: 'icon-telemetry',
        
        // Chỉ hiển thị view này cho type 'echarts-plot'
        canView(domainObject) {
            return domainObject.type === 'echarts-plot';
        },

        view(domainObject, objectPath) {
            let app = null;
            let componentInstance = null;

            return {
                show(element) {
                    // Mount component Vue vào DOM
                    app = createApp(EchartsComponent, {
                        openmct,
                        domainObject
                    });
                    componentInstance = app.mount(element);
                },
                destroy() {
                    if (app) {
                        app.unmount();
                        app = null;
                    }
                }
            };
        }
    };
}
```

---

## 3. File Component lõi: EchartsComponent.vue
Đây là nơi phép thuật xảy ra. Chúng ta sẽ "bắt" logic lấy dữ liệu của Open MCT và "bơm" vào ECharts.

```vue
<template>
    <div class="echarts-wrapper" style="width: 100%; height: 100%;">
        <!-- Container cho ECharts -->
        <div ref="chartContainer" style="width: 100%; height: 100%;"></div>
    </div>
</template>

<script>
// Nhớ cài đặt echarts qua npm: npm install echarts
import * as echarts from 'echarts';

export default {
    props: {
        openmct: { type: Object, required: true },
        domainObject: { type: Object, required: true }
    },
    data() {
        return {
            chartInstance: null,
            telemetryObjects: [],
            subscriptions: [],
            seriesData: {} // Lưu data theo dạng { 'id-của-sensor': [[x1, y1], [x2, y2]] }
        };
    },
    mounted() {
        this.initChart();
        this.loadComposition();
        
        // Lắng nghe sự kiện resize (Rất quan trọng cho ECharts)
        window.addEventListener('resize', this.resizeChart);
    },
    beforeUnmount() {
        // Dọn dẹp subscriptions và instance khi đóng Component
        this.subscriptions.forEach(unsubscribe => unsubscribe());
        if (this.composition) {
            this.composition.off('add', this.onTelemetryAdded);
        }
        window.removeEventListener('resize', this.resizeChart);
        if (this.chartInstance) {
            this.chartInstance.dispose();
        }
    },
    methods: {
        initChart() {
            // Khởi tạo ECharts
            this.chartInstance = echarts.init(this.$refs.chartContainer, 'dark'); // Hỗ trợ dark mode có sẵn
            this.chartInstance.setOption({
                tooltip: { trigger: 'axis' },
                xAxis: { 
                    type: 'time',
                    name: 'Thời gian',
                    splitLine: { show: false }
                },
                yAxis: { 
                    type: 'value',
                    name: 'Giá trị'
                },
                series: []
            });
        },
        
        loadComposition() {
            // Lấy danh sách các object con được user kéo thả vào Plot
            this.composition = this.openmct.composition.get(this.domainObject);
            if (this.composition) {
                this.composition.on('add', this.onTelemetryAdded);
                this.composition.load(); // Kích hoạt sự kiện 'add' cho các object đã lưu
            }
        },

        onTelemetryAdded(telemetryObj) {
            const id = this.openmct.objects.makeKeyString(telemetryObj.identifier);
            this.telemetryObjects.push(telemetryObj);
            
            // Khởi tạo buffer chuẩn bị nhận data cho Series này
            this.seriesData[id] = [];
            
            // Khởi tạo series rỗng trên ECharts
            this.updateEchartsSeries(id, telemetryObj.name);

            // BƯỚC QUAN TRỌNG NHẤT 1: Request Dữ Liệu Lịch Sử (Historical Data)
            const timeCtx = this.openmct.time.getContextForView([]);
            const bounds = timeCtx.getBounds();
            
            this.openmct.telemetry.request(telemetryObj, {
                start: bounds.start,
                end: bounds.end
            }).then(historicalData => {
                historicalData.forEach(datum => this.processDatum(id, telemetryObj, datum));
                this.refreshChart(); // Render 1 cục historical
            });

            // BƯỚC QUAN TRỌNG NHẤT 2: Subscribe Dữ Liệu Thực Tế (Realtime Data)
            const unsubscribe = this.openmct.telemetry.subscribe(telemetryObj, (realtimeDatum) => {
                this.processDatum(id, telemetryObj, realtimeDatum);
                this.refreshChart(); // Render từng điểm realtime
            });

            this.subscriptions.push(unsubscribe);
        },

        processDatum(id, telemetryObj, datum) {
            // Lấy metadata để biết trường nào là X (thời gian), Y (giá trị)
            const metadata = this.openmct.telemetry.getMetadata(telemetryObj);
            const valueKey = metadata.valuesForHints(['range'])[0].source;
            const timeKey = metadata.valuesForHints(['domain'])[0].source;

            const x = datum[timeKey];
            const y = datum[valueKey];

            // ECharts format: [ [x, y], [x, y] ]
            this.seriesData[id].push([x, y]);
            
            // (Tùy chọn) Shift data cũ đi nếu quá 10,000 điểm để tối ưu hiệu năng
            if (this.seriesData[id].length > 10000) {
                this.seriesData[id].shift();
            }
        },

        updateEchartsSeries(id, name) {
            // Cập nhật cấu hình ECharts để tạo Line đẹp mắt
            const currentOption = this.chartInstance.getOption();
            const series = currentOption.series || [];
            
            series.push({
                id: id,
                name: name,
                type: 'line',
                smooth: true,               // Đường cong mượt
                symbol: 'none',             // Ẩn dấu chấm để đỡ rối mắt
                lineStyle: { width: 3 },
                areaStyle: {                // Thêm hiệu ứng gradient bóng đổ bên dưới đẹp mắt
                    opacity: 0.2
                },
                data: []
            });

            this.chartInstance.setOption({ series });
        },

        refreshChart() {
            // Đẩy toàn bộ data mới vào ECharts
            const seriesUpdate = Object.keys(this.seriesData).map(id => ({
                id: id,
                data: this.seriesData[id]
            }));

            this.chartInstance.setOption({
                series: seriesUpdate
            });
        },

        resizeChart() {
            if (this.chartInstance) {
                this.chartInstance.resize();
            }
        }
    }
};
</script>
```

---

## Tóm tắt những điểm cần lưu ý với ECharts x Open MCT:

1. **Dark/Light Mode:** Open MCT mặc định nền tối. Khởi tạo `echarts.init(..., 'dark')` sẽ giúp chart trông như "native" của hệ thống sinh ra.
2. **Hiệu năng:** Code ở trên giữ buffer [seriesData](file:///d:/work/openmct/openmct/src/plugins/plot/MctPlot.vue#341-344) 10,000 điểm ảnh. Với ECharts, nếu bạn muốn vẽ nhiều hơn, bạn nhớ tìm hiểu về thuộc tính `sampling: 'lttb'` trong config của Echarts series để nó tự nội suy các điểm.
3. **Smooth Line:** Thêm tùy chọn `smooth: true` và `areaStyle` như ví dụ trên sẽ khắc phục được nhược điểm đường gấp khúc khô khan và cũ kĩ của Open MCT mặc định.

Bạn có thể lưu các đoạn script này lại và đưa vào `/src/plugins/MyEchartsPlugin` và thêm `openmct.install(MyEchartsPlugin())` ở index.html (hoặc file entry app) để tự chạy màn hình này.
