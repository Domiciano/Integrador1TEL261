# Gráficos

Si tiene almacenadas series de tiempo en su proyecto, probablemente quiera o tenga que graficarlos.

Para conseguirlo, puede usar cualquier librería gráfica presente en NPM.

En este ejemplo se usará MUI Charts

Para instalarlo

```bash
npm install @mui/x-charts
```

Posteriormente, puede usar este ejemplo para adaptar sus datos a la gráfica

```jsx
import { Box } from "@mui/material";
import { LineChart } from "@mui/x-charts";

const Graph = () => {
  const tiempo = [0, 1, 2, 3, 4, 5, 6, 7]; 
  const magnitud = [1, 4, 2, 5, 7, 2, 4, 60]; 

  return (
    <Box sx={{ width: 500, height: 300 }}>
      <LineChart
        xAxis={[
          {
            data: tiempo,
            label: 'Tiempo',
          },
        ]}
        yAxis={[
          {
            label: 'Magnitud',
          },
        ]}
        series={[
          {
            data: magnitud,
            label: 'Señal',
            showMark: false,
          },
        ]}
        grid={{ horizontal: true, vertical: true }}
      />
    </Box>
  );
};

export default Graph;
```

🎯 Cree un endpoint GET que permita descargar una muestra

🎯 Defina la forma de los datos para que el componente web descargue la información ya de una vez lista para graficar

🎯 Obtenga los datos, páselos al gráfico y observe la señal (Escoja una de las 6 señales)


